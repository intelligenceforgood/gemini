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

# ML Training Workflow Standards

## Training Container Authoring
- **Coerce feature dtypes before XGBoost.** JSONL features can arrive as strings even when they look numeric. After building a feature DataFrame, always coerce: `df.apply(pd.to_numeric, errors="coerce").fillna(0)`. XGBoost's `DMatrix` rejects `object` dtype columns with a cryptic `KeyError: 'object'`.
- **Pin `classification_report` to all encoder classes.** When some classes have zero eval samples, `classification_report(y_true, y_pred, target_names=le.classes_)` raises `ValueError: Number of classes N does not match size of target_names M`. Always pass both extra params.

## Pipeline Submission
- **Upload the config to GCS before submitting.** If `submit_pipeline.py` constructs a `gs://` path for a config parameter, the training container will 404 unless the local file is actually uploaded first.

## Debugging Loop — Test Locally First
**Never iterate via cloud submissions.** Each build→push→submit→wait cycle takes ~10 minutes. Instead:
1. Download a small sample once.
2. Run the script directly locally.
3. Only build+push+submit once the local run exits cleanly.

## Monitoring Submitted Jobs
Poll pipeline state via the Python SDK. To get the inner CustomJob ID from a failed pipeline, check the outer job logs for the `resource.labels.job_id` in the error URL, then pull errors with:
```bash
gcloud logging read 'resource.type="ml_job" resource.labels.job_id="<id>" severity>=DEFAULT' \
  --project=i4g-ml --format='value(timestamp, textPayload)' --order=asc --limit=500 2>&1 \
  | grep -A5 -E "Traceback|Error:|raise "
```