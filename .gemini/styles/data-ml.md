---
applyTo: "**/*.sql,**/etl*.py,**/bigquery/**,ml/containers/train-*/**,ml/scripts/submit_*.py,ml/src/ml/training/**"
---

# BigQuery & ETL Standards

## SQL Files

- **Never break backtick-quoted identifiers across lines.** Formatters can split `` `project.dataset.table` `` onto multiple lines, causing `Unclosed identifier literal` errors. Keep all backtick identifiers on a single line.
- **`JSON_EXTRACT` path must be a constant expression.** `JSON_EXTRACT(col, CONCAT('$.', var))` fails. Use individual checks: `IF(JSON_EXTRACT(col, '$.key') IS NOT NULL, 1, 0)`.

## ETL Data Coercion (pg8000 → BigQuery)

- **pg8000 returns `Decimal` for numeric columns.** Always coerce before `load_table_from_json`: `Decimal → float`, `datetime → isoformat()`, `dict/list → json.dumps(default=str)`.
- **Postgres JSONB → BQ STRING, not JSON type.** Serialize JSONB to Python string with `json.dumps()` and store as BQ `STRING`. BQ `JSON` type causes autodetect to infer `STRUCT`, which breaks MERGE.

## Staging Tables

- **Never use `autodetect=True` for staging tables.** Autodetect infers types from data (e.g., `"1.0"` → FLOAT64 instead of STRING), then MERGE fails on type mismatch. Always use the target table's schema: `bq_client.get_table(target).schema`.

## Schema Changes

- **BQ tables need delete + recreate after significant schema changes.** Terraform can't remove or rename columns in-place. `bq rm -f` the tables, then `terraform apply` to recreate.
- **BigQuery dataset `access` blocks replace all defaults.** Always include the three default `special_group` entries (`projectOwners`, `projectWriters`, `projectReaders`) alongside custom grants.

---
applyTo: "ml/containers/train-*/**,ml/scripts/submit_*.py,ml/src/ml/training/**"
---

# ML Training Workflow Standards

## Training Container Authoring

- **Coerce feature dtypes before XGBoost.** JSONL features can arrive as strings even when they look numeric. After building a feature DataFrame, always coerce: `df.apply(pd.to_numeric, errors="coerce").fillna(0)`. XGBoost's `DMatrix` rejects `object` dtype columns with a cryptic `KeyError: 'object'`.

- **Pin `classification_report` to all encoder classes.** When some classes have zero eval samples, `classification_report(y_true, y_pred, target_names=le.classes_)` raises `ValueError: Number of classes N does not match size of target_names M`. Always pass both extra params:
  ```python
  classification_report(
      y_eval, y_pred,
      labels=list(range(len(le.classes_))),
      target_names=le.classes_,
      zero_division=0,
  )
  ```

## Pipeline Submission

- **Upload the config to GCS before submitting.** If `submit_pipeline.py` constructs a `gs://` path for a config parameter, the training container will 404 unless the local file is actually uploaded first:
  ```python
  bucket.blob(f"configs/{Path(config_path).name}").upload_from_filename(config_path)
  ```

## Debugging Loop — Test Locally First

**Never iterate via cloud submissions.** Each build→push→submit→wait cycle takes ~10 minutes. Instead:

1. Download a small sample once:
   ```bash
   mkdir -p /tmp/xgb-test/data
   gsutil cat gs://i4g-ml-data/datasets/classification/v1/train.jsonl | head -100 > /tmp/xgb-test/data/train.jsonl
   gsutil cat gs://i4g-ml-data/datasets/classification/v1/eval.jsonl  | head -30  > /tmp/xgb-test/data/eval.jsonl
   gsutil cat gs://i4g-ml-data/datasets/classification/v1/test.jsonl  | head -30  > /tmp/xgb-test/data/test.jsonl
   cp pipelines/configs/classification_xgboost.yaml /tmp/xgb-test/
   ```

2. Run the script directly (~2s feedback):
   ```bash
   conda run -n ml python containers/train-xgboost/train.py \
     --config /tmp/xgb-test/classification_xgboost.yaml \
     --dataset /tmp/xgb-test/data \
     --output /tmp/xgb-test/output
   ```

3. Only build+push+submit once the local run exits cleanly.

## Monitoring Submitted Jobs

Poll pipeline state via the Python SDK (gcloud CLI doesn't have `pipeline-jobs describe`):
```python
from google.cloud import aiplatform
aiplatform.init(project="i4g-ml", location="us-central1")
job = aiplatform.PipelineJob.get("<resource_name>")
print(job.state)  # 3=RUNNING, 4=SUCCEEDED, 5=FAILED
```

To get the inner CustomJob ID from a failed pipeline, check the outer job logs for the `resource.labels.job_id` in the error URL, then pull errors with:
```bash
gcloud logging read 'resource.type="ml_job" resource.labels.job_id="<id>" severity>=DEFAULT' \
  --project=i4g-ml --format='value(timestamp, textPayload)' --order=asc --limit=500 2>&1 \
  | grep -A5 -E "Traceback|Error:|raise "
```

## SQL / Database

- **Table and column names:** `snake_case`, singular (`review_queue`, not
  `ReviewQueues`).
- **Migrations:** Use Alembic. Each migration gets a descriptive message.
  Never modify a migration that has been applied to a shared environment.
- **Queries:** Use SQLAlchemy ORM or Core expressions. Raw SQL only when ORM
  cannot express the query; always use parameterized queries (never string
  interpolation).

---

## 6. Database Schema (Key Tables)

### Core tables (`src/i4g/store/sql.py`, METADATA)

- `cases` — Central entity, one per scam report. PK: `case_id`. Carries classification, status, tags, metadata JSON.
- `source_documents` — Evidence chunks. FK: `case_id`. Has `document_id` (UUID), `title`, `mime_type`, `source_url`.
- `review_queue` — Analyst work queue. Links to `case_id`. Has priority, status, assignment, classification results.
- `review_actions` — Audit log. FK: `review_id`. Stores `actor`, `action`, `payload` (JSON), `created_at`.
- `indicators` — Fraud indicators (wallets, phones, URLs). FK: `case_id`. Typed with confidence scores.
- `entities` — Extracted entities (persons, emails). FK: `case_id`.
- `saved_searches` — Persisted analyst queries.
- `dossier_queue` — Report generation tasks.
- `intake_records/attachments/jobs` — Victim submission pipeline.

### SSI tables (shared DB — defined in both `core/src/i4g/store/sql.py` and `ssi/src/ssi/store/sql.py`)

- `site_scans` — Investigation metadata. PK: `scan_id`. Has `case_id` FK, `evidence_path`, `scan_status`, `risk_score`, `classification` JSON.
- `harvested_wallets` — Wallet addresses. FKs: `case_id`, `scan_id`.
- `agent_sessions` — Per-action audit trail for browser agent.
- `pii_exposures` — What PII the scam site collects.

### Key relationships

```
cases ──1:N──▶ source_documents (evidence files)
cases ──1:N──▶ indicators (wallets, IPs, etc.)
cases ──1:N──▶ entities (persons, emails)
review_queue ──1:N──▶ review_actions (timeline)
cases ──1:1──▶ site_scans (SSI investigation)
site_scans ──1:N──▶ harvested_wallets
```
