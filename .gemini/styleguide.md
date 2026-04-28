---
applyTo: "**/*.py"
---

# Python Standards

- **Type hints:** Required on all functions — parameters + return type.
- **Docstrings:** Google-style. Required on public functions, classes, modules. Private helpers (`_name`) get at least a one-liner.
- **Formatter:** Black at 120 chars. isort (profile = black).
- **Naming:** `snake_case` functions/variables/methods/files. `PascalCase` classes. `UPPER_SNAKE_CASE` constants. Singular nouns for constants and config keys.
- **Pydantic:** `snake_case` fields in Python. Use `CamelModel` with `alias_generator = to_camel` for JSON output — never write manual casing functions.
- **Imports:** Absolute preferred (`from i4g.store.review_store import ReviewStore`). Relative only within same sub-package. No wildcards.
- **Error handling:** Catch specific exceptions. Never bare `except:`. Log with `logger.exception()`.
- **Datetime:** Use `datetime.now(datetime.UTC)`, never `datetime.utcnow()`.
- **Logging:** `logger = logging.getLogger(__name__)` per module. Structured key-value pairs.
- **Settings:** Always via `get_settings()`, not hard-coded values. Override via env vars, not code.
- **Stores:** Use factories (e.g., `factories.py`) — do not instantiate stores directly.
- **TYPE_CHECKING guard:** For expensive runtime-only imports used solely as types (e.g., Playwright `Page`), guard with `if TYPE_CHECKING:`.
---
applyTo: "**/*.ts,**/*.tsx,**/*.jsx"
---

# TypeScript / React Standards

- **TypeScript required** for all new code. No `.js` files except build configs.
- **Formatter:** Prettier via `pnpm format`. Always run after editing.
- **Naming:** `camelCase` for variables/functions/properties/hooks. `PascalCase` for components/classes/types/interfaces/enum members. `UPPER_SNAKE_CASE` for constants.
- **File names:** `kebab-case.ts` for utilities, `PascalCase.tsx` for React components (follow directory convention).
- **Types:** Interfaces over `type` aliases for object shapes. Discriminated unions for status branching. Interfaces mirror API schemas with camelCase field names.
- **React:** Functional components with hooks only. `React.FC` only when children needed. Derived state → custom hooks. Style via CSS modules or design system.
- **Exports:** Named exports preferred. Default exports only for Next.js pages/layouts.
- **Error handling:** Never swallow errors. Use Error boundaries. `try/catch` must log or surface.
- **Null handling:** Prefer `undefined` over `null`. Use `?.` and `??`.
- **No manual casing translation.** Use framework-native serialization.
- **HTML/CSS:** Semantic HTML. Keyboard accessibility. `alt` on images. CSS Modules preferred. No `!important`.
---
applyTo: "**/*.tf,**/*.tfvars"
---

# Terraform / HCL Standards

- **Formatter:** `terraform fmt` before committing.
- **Naming:** `snake_case` for all resource names, variables, outputs, locals. Module directories: `snake_case` or `kebab-case` (match existing convention).
- **Variables:** Always include `description` and `type`. Use `default` only when a sensible safe default exists.
- **Sensitive values:** Mark with `sensitive = true`. Never put secrets in `.tfvars` — use Secret Manager.
- **Outputs:** Provide for any value downstream modules or the console need.
- **File layout per module:** `main.tf`, `variables.tf`, `outputs.tf`. Large modules split into logical files (`iam.tf`, `networking.tf`).
- **Target dev first.** Never apply to prod without a successful dev deployment.
- **State:** Use GCS remote state.
- **Auth:** Impersonate service account with `gcloud auth application-default login`.
---
applyTo: "**/settings*.toml,**/settings*.py,**/.env*,**/pyproject.toml"
---

# Settings & Configuration Standards

- **Settings access:** Always via `get_settings()` in code. Never hard-code paths, URLs, or credentials.
- **Environment overrides:** Use env vars with project prefix (e.g., `I4G_*`). Double underscores for nesting (`I4G_LLM__PROVIDER`).
- **TOML config:** `snake_case` keys. Section headers in `[brackets]`.
- **Env profiles:** `local` → SQLite/mock. `dev`/`prod` → PostgreSQL/cloud services.
- **Path resolution:** Use framework's path resolver — pass project-relative references, not manual `Path` math.
- **Env var contract:** When adding/changing env vars:
  1. Add test coverage under `tests/unit/settings/`
  2. Update config documentation/manifests
  3. Run local smoke test before cloud deployment
- **.env files:** `UPPER_SNAKE_CASE` keys. Non-public secrets go in `.env.local` (gitignored).
---
applyTo: "*"
---

# Merge & Commit Discipline

Hard-won rules for the commit-and-push phase of the merge routine. These apply
to **every** file edit made during or after a pre-merge review — not just the
main feature changes.

## 1. Format before staging (UI repo)

When editing JSON, TS, or TSX files via the editor or edit tool, the output
often diverges from Prettier's canonical formatting (array wrapping, trailing
newlines, indentation). Always run the formatter **before** `git add`:

```bash
cd ui/ && pnpm format        # Prettier writes canonical output
git add -A                   # now safe to stage
git commit -m "..."
```

Skipping this causes the committed content to differ from Prettier — the next
`pnpm format` run re-formats the file and leaves a dirty working tree that
looks like an uncommitted change.

## 2. Build before committing config changes

Any edit to `tsconfig.json`, `package.json`, build config, or settings files
must pass the full build **before** `git commit`:

```bash
# UI
cd ui/ && make build

# Python
cd core/ && conda run -n i4g pre-commit run --all-files
```

This catches breakage (e.g., removing `baseUrl` breaking path aliases) before
it reaches `origin/main`.

## 3. Post-push cleanliness sweep

After pushing, run `git status -sb` in **every** workspace repo — not just the
repos you changed. Formatters, editors, and hooks can silently modify files
after a commit. The merge is not done until all repos show a clean working tree.

```bash
for repo in copilot core ssi ui infra ml docs planning mobile; do
  dirty=$(cd /path/to/i4g/$repo && git status --porcelain 2>/dev/null)
  [[ -n "$dirty" ]] && echo "DIRTY: $repo" && echo "$dirty"
done
```

If any repo is dirty, diagnose whether it's a formatter artifact (commit it) or
an unintended change (revert it) before declaring the merge complete.
---
applyTo: "ui/packages/sdk/**,ui/apps/web/**"
---

# UI / SDK Build Checklist

Apply these checks whenever adding or modifying UI components, SDK types, or client methods.

## SDK Interface Sync

- When adding methods to the `I4GClient` interface in `packages/sdk/src/index.ts`, **always** add matching stubs to `createMockClient()` in `packages/sdk/src/__fixtures__/index.ts`. The build fails otherwise.
- When adding Zod schemas, add the corresponding `export type` and wire the schema into the relevant client method.

## Union Type Safety

- Before using a string literal as a component prop variant (e.g., `Badge variant=`), **check the component source** for the actual union type. Common mistake: using `"secondary"` when `BadgeVariant` only defines `"default" | "success" | "warning" | "danger" | "info"`.
- Never guess variant values — read the type definition.

## Zod Schema ↔ API Parity

- SDK Zod schemas must match what the FastAPI backend actually returns. `curl` the endpoint to verify field names and types before writing a new schema.
- After changing a Zod schema, search `packages/sdk/src/__fixtures__/` for affected field names and update test fixtures.

## Pre-Commit Verification

Before staging UI changes for merge, run in order:

1. `pnpm format` — Prettier formatting
2. `pnpm lint` — catches unused imports, undeclared vars
3. `pnpm build` — catches type errors, missing mock stubs, invalid union values

All three must pass. Do not rely on editor diagnostics alone — `pnpm build` is the source of truth.
---
applyTo: "**/*.sql,**/etl*.py,**/bigquery/**"
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
---
applyTo: "**/*.py"
---

# Ruff Pitfalls

Common ruff violations encountered in this codebase and how to handle them.

- **B008 — Typer `Option()`/`Argument()` in defaults.** Typer's documented pattern `def cmd(name: str = typer.Option(...))` triggers B008. Don't refactor — add `"src/*/cli/*.py" = ["B008"]` to `[tool.ruff.lint.per-file-ignores]` in pyproject.toml.
- **UP038 — `isinstance` with tuple of types.** `isinstance(x, (int, float))` → `isinstance(x, int | float)`. Use union syntax on Python 3.10+.
- **UP042 — `str, Enum` base classes.** `class Foo(str, Enum)` → `class Foo(StrEnum)` using `from enum import StrEnum` (Python 3.11+).
- **SIM105 — try/except/pass.** Replace with `contextlib.suppress(ExceptionType)`. Remember to `import contextlib`.
- **SIM102 — nested `if` statements.** Merge `if a: if b:` into `if a and b:` when the outer `if` has no `else`.
- **F841 — unused variable assignment.** Remove the assignment or prefix with `_` if the call has side effects (e.g., `_client = SomeClass()`).
- **UP037 — quoted type annotations.** Remove quotes from annotations when `from __future__ import annotations` is not needed (Python 3.11+ with union syntax).
- **E501 — line too long.** Break long `if` conditions into parenthesized multi-line format. Break long strings with implicit concatenation.
- **Redundant except clauses.** `except (SpecificError, Exception)` is redundant — `Exception` subsumes `SpecificError`. Catch the specific exception only.
# General Coding Guidelines

These are the shared coding standards referenced by `.github/copilot-instructions.md`
in every repository. **Follow the idiomatic conventions of each language/tool --
do not invent project-specific patterns when the ecosystem already has a clear
standard.**

## Universal Principles

- **Consistency over preference.** Match the style of the file and project you
  are editing. When writing new code, follow the language-specific rules below.
- **Configuration-driven code.** Fetch settings through
  `i4g.settings.get_settings()` and honor the environment-aware factories in
  `src/i4g/services/factories.py`. Hard-coded paths, URLs, and credentials are
  never acceptable.
- **ASCII by default.** Keep edits ASCII unless a file already depends on
  Unicode. Never revert user-authored changes without explicit direction.
- **Test before shipping.** Run relevant tests or smoke flows (`pytest
tests/unit`, targeted `tests/adhoc/` scripts, `i4g bootstrap local reset`)
  before shipping changes; note any skipped suites in summaries.
- **Docs in sync.** Material changes update `planning/change_log.md`,
  `docs/design/architecture.md`, or `docs/development/dev_guide.md` as needed.
- **No manual casing translation layers.** When data crosses a language boundary
  (e.g., Python API to TypeScript client), use framework-native serialization
  (Pydantic `alias_generator`, Zod `.transform()`, etc.) rather than writing
  manual field-rename functions. See D79 in `debt_remediation_plan.md`.

---

## Python

- **Type hints:** Required on all new or modified functions (parameters + return).
- **Docstrings:** Google-style. Required on public functions, classes, and
  modules. Concise explanatory comments when logic is non-obvious.
- **Formatter:** Black, 120-char line limit. Import ordering: isort (profile =
  black).
- **Naming:**
  - `snake_case` for functions, variables, methods, module file names.
  - `PascalCase` for classes and type aliases.
  - `UPPER_SNAKE_CASE` for module-level constants.
  - Singular nouns for constants, file names, database tables, and configuration
    keys (e.g., `REPORT_BUCKET`, not `REPORTS_BUCKET`). Avoid plurals unless the
    value is a collection.
- **Pydantic models:** Field names are `snake_case` in Python. When serving JSON
  to non-Python clients, use a `CamelModel` base with `alias_generator = to_camel`
  and `by_alias=True` so the wire format is camelCase without manual mapping.
- **Imports:** Absolute imports preferred (`from i4g.store.review_store import
ReviewStore`). Relative imports only within the same sub-package.
- **Error handling:** Catch specific exceptions. Never bare `except:`. Log
  errors with `logger.exception()` or `logger.error(..., exc_info=True)`.
- **Datetime:** Use `datetime.now(datetime.UTC)`, never `datetime.utcnow()`
  (deprecated Python 3.12+).
- **Logging:** Every module: `logger = logging.getLogger(__name__)`. Use
  structured key-value pairs in log messages (`logger.info("action", extra={...})`
  or f-string with context).

## TypeScript / JavaScript

- **TypeScript required** for all new code. No `.js` files in `ui/` unless
  they are build configs (e.g., `next.config.js`, `tailwind.config.js`).
- **Formatter:** Prettier via `pnpm format`. Always run after editing.
- **Linter:** ESLint with the project config. Zero warnings in CI.
- **Naming:**
  - `camelCase` for variables, functions, object properties, React hooks.
  - `PascalCase` for components, classes, type/interface names, enum members.
  - `UPPER_SNAKE_CASE` for true constants and environment variable references.
  - File names: `kebab-case.ts` for utilities, `PascalCase.tsx` for React
    components (follow whichever convention the enclosing directory already uses).
- **Types:** Favor interfaces over `type` aliases for object shapes. Use
  discriminated unions for variant/status handling. Model data with interfaces
  that mirror FastAPI schemas (camelCase field names matching the Pydantic
  `alias_generator` output).
- **React:** Functional components with hooks only. Use `React.FC` only when
  children are required. Move derived state to custom hooks. Style via CSS
  modules or `@i4g/ui-kit`.
- **Exports:** Named exports preferred. Default exports only for Next.js page
  and layout files (required by framework).
- **Error handling:** Never swallow errors silently. `try/catch` blocks must
  log or surface the error. Use Error boundaries in React.
- **Null handling:** Prefer `undefined` over `null` in new code (unless
  matching an external API contract). Use optional chaining (`?.`) and
  nullish coalescing (`??`).

## HTML / CSS

- **Semantic HTML.** Use `<main>`, `<nav>`, `<section>`, `<article>`, `<button>`
  -- not `<div>` with click handlers.
- **Accessibility.** All interactive elements must be keyboard-reachable. Images
  need `alt` text. Form inputs need `<label>`.
- **CSS Modules** preferred in `ui/`. Avoid inline styles. Use design tokens
  from `@i4g/ui-kit` or `mobile/shared/design-tokens` when available.
- **No `!important`** unless overriding third-party CSS with no other option.

## Terraform / HCL

- **Formatter:** `terraform fmt`. Run before committing.
- **Naming:**
  - `snake_case` for all resource names, variable names, output names, locals.
  - Module directories: `snake_case` or `kebab-case` (match existing convention
    in `infra/modules/`).
- **Variables:** Always include `description` and `type`. Use `default` only
  when a sensible safe default exists; required variables have no default.
- **Sensitive values:** Mark with `sensitive = true`. Never put secrets in
  `.tfvars` -- use Secret Manager references.
- **Outputs:** Provide outputs for any value that downstream modules or the
  console need.
- **File layout per module:** `main.tf`, `variables.tf`, `outputs.tf`. Large
  modules may split into logical files (e.g., `iam.tf`, `networking.tf`).
- **Target `i4g-dev` first.** Never apply to `i4g-prod` without a successful
  dev deployment.

## TOML / YAML / JSON (Config Files)

- **TOML** (`settings.toml`, `pyproject.toml`): `snake_case` keys. Section
  headers in `[brackets]`.
- **YAML** (GitHub Actions, manifests): `snake_case` or `kebab-case` depending
  on the tool's convention. Prefer block scalars over long single-line strings.
- **JSON** (package.json, tsconfig): Follow the tool's required key names.
  Keep formatting consistent with Prettier (`pnpm format`).
- **`.env` files:** `UPPER_SNAKE_CASE` keys. Prefix project env vars with
  `I4G_`. Use double underscores for nested sections (`I4G_LLM__PROVIDER`).

## SQL / Database

- **Table and column names:** `snake_case`, singular (`review_queue`, not
  `ReviewQueues`).
- **Migrations:** Use Alembic. Each migration gets a descriptive message.
  Never modify a migration that has been applied to a shared environment.
- **Queries:** Use SQLAlchemy ORM or Core expressions. Raw SQL only when ORM
  cannot express the query; always use parameterized queries (never string
  interpolation).

## Shell / Bash

- **Use `set -euo pipefail`** at the top of scripts.
- **Quote all variables:** `"$var"`, not `$var`.
- **Naming:** `snake_case` for local variables, `UPPER_SNAKE_CASE` for
  exported env vars.
- **Shebang:** `#!/usr/bin/env bash`.
- **Portability:** Prefer POSIX-compatible constructs. Use `[[ ]]` for
  conditionals and `$()` for command substitution (not backticks).

## Markdown / Documentation

- **Grammar:** Present tense ("is", "runs"), active voice, second person ("you").
  Write factual statements; avoid "could"/"would".
- **Structure:** Use headings, bullet points, and code blocks. Keep lines at most
  120 chars where practical.
- **Code snippets:** Do NOT paste entire source files. Show focused excerpts
  (3-15 lines) with a link to the full file path.

---

## Collaboration & Review

- Treat collaboration as a two-person team (you + Copilot); you own
  prioritization and approvals.
- When preparing to merge, request (or expect) a "picky reviewer" pass: verify
  style conformance (including `pnpm format` for UI, `terraform fmt` for infra),
  remove dead code, ensure tests/docs reflect behavior across all repositories,
  and confirm deployment instructions (e.g., Cloud Run) are accurate.
# Architecture Cheat Sheet

Dense reference for the i4g multi-root workspace. Absorb before every coding session to avoid wrong assumptions about routing, auth, data flow, and storage.

---

## 1. Request Routing (UI → Core → SSI)

### Three-layer proxy chain

```
Browser (port 3000)
  ↓  href="/api/..."  or  SDK fetch("/api/...")
Next.js (App Router, port 3000)
  ↓  server-side fetch to I4G_API_URL (default http://127.0.0.1:8000)
Core FastAPI (port 8000)
  ↓  (for SSI) HTTP POST to SSI Cloud Run Service
SSI FastAPI (port 8100 locally; deployed as Cloud Run Service `ssi-svc` in cloud)
```

### Next.js API routes — two patterns

| Pattern              | Location                                                               | Purpose                                                                                                                             |
| -------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Dedicated routes** | `ui/apps/web/src/app/api/ssi/*`, `api/search/*`, `api/reviews/*`, etc. | Custom proxy logic (request normalization, response shaping, error handling). Always preferred when they exist.                     |
| **Catch-all proxy**  | `ui/apps/web/src/app/api/[...path]/route.ts`                           | Generic pass-through to core API. Forwards any `GET/POST/PUT/DELETE/PATCH` to `{I4G_API_URL}/{path}`. No special response handling. |

### Critical URL rule for browser-facing links

**Core API returns API-relative paths** (e.g., `/cases/{id}/evidence/{doc_id}`).
**Browser links MUST be prefixed with `/api`** so they route through the Next.js proxy.
Never generate absolute `http://localhost:8000/...` URLs in API responses — the browser cannot reach the core API directly in cloud.

```
Core returns:  url="/cases/{id}/evidence/{doc_id}"
UI renders:    href="/api/cases/{id}/evidence/{doc_id}"
                     ^^^^  ← prefix added by UI
Browser hits:  localhost:3000/api/cases/{id}/evidence/{doc_id}
Proxy routes:  → localhost:8000/cases/{id}/evidence/{doc_id}
```

### SDK client URL resolution

| Context                    | baseUrl                                                   | Mechanism                                       |
| -------------------------- | --------------------------------------------------------- | ----------------------------------------------- |
| Server component (SSR)     | `process.env.I4G_API_URL` (e.g., `http://127.0.0.1:8000`) | Direct to core — no proxy needed                |
| Client component (browser) | `"/api"`                                                  | Relative — goes through Next.js catch-all proxy |

Source: `ui/apps/web/src/lib/i4g-client.ts` → `resolveClient()`.

### SSI-specific Next.js API routes

| Browser path                   | Next.js route file                     | Proxies to (core API)                                      | Notes                                               |
| ------------------------------ | -------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------- |
| `/api/ssi/investigate`         | `api/ssi/investigate/route.ts`         | Core `POST /investigations/ssi`                            | Core orchestrates; triggers SSI service              |
| `/api/ssi/investigate/{id}`    | `api/ssi/investigate/[id]/route.ts`    | Core `GET /tasks/{id}`                                     | Core is single source of truth for task status       |
| `/api/ssi/report/{id}`         | `api/ssi/report/[id]/route.ts`         | Core `GET /investigations/ssi/{id}/report.pdf`                         | Handles GCS 307 redirects, sets Content-Disposition |
| `/api/ssi/investigations`      | `api/ssi/investigations/route.ts`      | Core `GET /investigations/ssi/history`                                 | Passes query params                                 |
| `/api/ssi/investigations/{id}` | `api/ssi/investigations/[id]/route.ts` | Core `GET /investigations/ssi/{id}`                                    | Direct proxy                                        |
| `/api/ssi/wallets`             | `api/ssi/wallets/route.ts`             | Core `GET /investigations/ssi/wallets`                                 | Direct proxy                                        |

### eCX proxy routes (direct to SSI — require `SSI_API_URL`)

The eCX proxy routes forward **directly to ssi-svc** (not through core). They use `resolveSsiUrl()` from `ui/apps/web/src/lib/server/ssi-proxy.ts`, which reads `SSI_API_URL`. In cloud, this env var is injected via Terraform (`module.run_ssi_service[0].uri`). Auth is handled by `ssiHeaders()` (OIDC identity token with ssi-svc URL as audience).

| Browser path                          | Next.js route file                                    | Proxies to (SSI)                       |
| ------------------------------------- | ----------------------------------------------------- | -------------------------------------- |
| `/api/ssi/ecx/feed`                   | `api/ssi/ecx/feed/route.ts`                           | SSI `GET /ecx/feed`                    |
| `/api/ssi/ecx/polling-status`         | `api/ssi/ecx/polling-status/route.ts`                 | SSI `GET /ecx/polling-status`          |
| `/api/ssi/ecx/submissions`            | `api/ssi/ecx/submissions/route.ts`                    | SSI `GET /ecx/submissions`             |
| `/api/ssi/ecx/submissions/{id}/approve` | `api/ssi/ecx/submissions/[id]/approve/route.ts`     | SSI `POST /ecx/submissions/{id}/approve` |
| `/api/ssi/ecx/submissions/{id}/reject`  | `api/ssi/ecx/submissions/[id]/reject/route.ts`      | SSI `POST /ecx/submissions/{id}/reject`  |
| `/api/ssi/ecx/submissions/{id}/retract` | `api/ssi/ecx/submissions/[id]/retract/route.ts`     | SSI `POST /ecx/submissions/{id}/retract` |
| `/api/ssi/ecx/investigate/{id}`       | `api/ssi/ecx/investigate/[id]/route.ts`               | SSI `POST /ecx/investigate/{id}`       |
| `/api/ssi/ecx/stats/*`                | `api/ssi/ecx/stats/*/route.ts`                        | SSI `GET /ecx/stats/*`                 |

### Core API router mounts (prefix map)

| Prefix                 | Router file             | Key endpoints                                                             |
| ---------------------- | ----------------------- | ------------------------------------------------------------------------- |
| `/cases`               | `cases.py`              | CRUD, timeline, detail                                                    |
| `/cases/{id}/evidence` | `evidence.py`           | Upload, list, download evidence files                                     |
| `/investigations/ssi`  | `ssi_investigations.py` | `/history`, `/active`, `/{scan_id}`                                       |
| `/investigations/ssi`  | `ssi_evidence.py`       | `/{scan_id}/report.pdf`, `/evidence-bundle`, `/lea-package`               |
| `/investigations/ssi`  | `ssi_wallets.py`        | `/wallets`, `/{scan_id}/wallets.csv`, `/{scan_id}/wallets.xlsx`           |
| `/investigations/ssi`  | `ssi_playbooks.py`      | Playbook CRUD (prefix: `/playbooks/ssi`)                                  |
| `/investigations`      | `investigations.py`     | `POST /investigations/ssi` (trigger), `GET /investigations/ssi/{task_id}` |
| `/reviews`             | `review.py`             | Search, history, saved searches, case actions                             |
| `/tasks`               | `tasks.py`              | `GET /tasks/{task_id}` — poll background task status                      |
| `/taxonomy`            | `taxonomy.py`           | Fraud taxonomy CRUD                                                       |

**Registration order matters** — SSI wallets/evidence routers are registered before `ssi_investigations` to prevent the `/{scan_id}` catch-all from swallowing `/wallets`, etc.

---

## 2. Auth Model

### Local environment (`I4G_ENV=local`)

**All auth is bypassed.** `require_token()` returns `{"username": "local-dev", "role": "admin"}` without checking any header. No API key, no JWT, no IAP. Do NOT debug 404s or 500s as auth issues in local env.

### Cloud environment (`I4G_ENV=dev` or `prod`)

- **IAP (Identity-Aware Proxy)**: Fronts Cloud Run services. Browser → IAP → Cloud Run.
- **API Key**: `X-API-KEY` header validated by `require_token()`.
- **JWT**: Google Identity Platform OIDC tokens validated by `require_token()`.
- **Next.js**: Server-side API routes inject IAP headers via `getIapHeaders()` before proxying to core.

### Key rule

**Never assume auth is the cause of errors in local dev.** If a request returns 404 or 500 locally, the issue is routing, missing data, or code bugs — not authentication.

---

## 3. Storage & Evidence

### Storage backends by environment

| Category      | Local                        | Cloud                |
| ------------- | ---------------------------- | -------------------- |
| Relational    | SQLite (`data/i4g_store.db`) | Cloud SQL PostgreSQL |
| Vector        | Chroma (`data/chroma_store`) | Vertex AI Search     |
| Blob/evidence | Local FS (`data/evidence/`)  | GCS buckets          |
| SSI scans     | Same shared DB (`i4g_store.db`) | Same Cloud SQL instance |

### Evidence file flow

```
SSI orchestrator → writes to data/evidence/{case_id}/
  ├── report.pdf, investigation.json, screenshots, etc.
  └── Updates site_scans.evidence_path = local path or gs:// URI

Core evidence endpoints:
  GET /investigations/ssi/{scan_id}/report.pdf
    → Local: serves from evidence_path/report.pdf
    → Cloud: 307 redirect to GCS signed URL

  GET /cases/{case_id}/evidence/{doc_id}
    → Serves from EvidenceStorage (abstracts FS vs GCS)
    → Requires source_documents row with matching doc_id
```

### SSI evidence storage layout

```
data/evidence/{case_id}/
  ├── investigation.json, report.md, report.pdf
  ├── stix_bundle.json, wallet_manifest.json, evidence.zip
  ├── passive/ (screenshot.png, dom.html, network.har, whois.json, etc.)
  └── active/  (session_log.json, screenshots/, wallets/)
```

---

## 4. LLM Auth (Gemini API)

### Auth modes (google-genai SDK)

Both Core and SSI use the `google-genai` SDK to call Gemini models. Two auth modes exist:

| Mode            | Setting                     | SDK call                                     | Billing target                        |
| --------------- | --------------------------- | -------------------------------------------- | ------------------------------------- |
| **API key**     | `gemini_api_key` is set     | `genai.Client(api_key=key)`                  | GCP project that owns the API key     |
| **Vertex AI**   | `gemini_api_key` is empty   | `genai.Client(vertexai=True, project=..., location=...)` | Vertex AI service in the GCP project  |

**API-key auth is preferred in cloud.** It routes billing through the non-profit's GCP billing account and avoids AI Studio cross-billing issues.

### Secret Manager wiring

| Service    | Secret name       | Env var injected by Terraform         |
| ---------- | ----------------- | ------------------------------------- |
| Core       | `gemini-api-key`  | `I4G_LLM__GEMINI_API_KEY`            |
| SSI        | `gemini-api-key`  | `SSI_LLM__GEMINI_API_KEY`            |

### Creating a Gemini API key

One key serves both Core and SSI (bound to `sa-app`, shared via Secret Manager).

1. Go to [GCP Console → APIs & Services → Credentials](https://console.cloud.google.com/apis/credentials) for the target project.
2. **Create Credentials → API key**.
3. Enter name (e.g., `gemini-api-key-dev`).
4. Check **Authenticate API calls through a service account** → select `sa-app@<PROJECT_ID>.iam.gserviceaccount.com`.
5. Under **Select API restrictions**, select **Gemini API** (appears after step 4).
6. Click **Create** and copy the key value.
7. Store in Secret Manager:
   ```bash
   echo -n "<YOUR_API_KEY>" | gcloud secrets versions add gemini-api-key \
     --data-file=- --project=<PROJECT_ID>
   ```

> Full details (rotation, troubleshooting): `core/docs/runbooks/gemini_api_key.md`.

### Fallback behavior

If `gemini_api_key` is empty/None, the code falls back to Vertex AI Application Default Credentials (`vertexai=True`). This still works but routes through `aiplatform.googleapis.com` instead of `generativelanguage.googleapis.com`.

---

## 5. SSI ↔ Core Integration

### Direct Database Writes (`ssi/src/ssi/store/scan_store.py`)

SSI writes investigation results directly to the shared database via `ScanStore.create_case_record()`:

```
create_case_record(scan_id, result, dataset)
  ├── INSERT cases            (case record + description + dedup via content hash)
  ├── INSERT scam_records     (search cache — no classification_result/tags)
  ├── INSERT review_queue     (analyst queue entry)
  ├── _insert_timeline_events()   (review_actions rows)
  ├── _insert_evidence_documents() (source_documents rows)
  └── UPDATE site_scans       (link scan → case)
```

### Case metadata from SSI

When `ScanStore` creates a case, it stores `ssi_investigation_id` in case metadata:

```python
"metadata": {
    "ssi_investigation_id": str(scan_id),
    "scan_type": result.scan_type,
    ...
}
```

This ID links the case back to SSI's `site_scans` table and is used to construct evidence download URLs.

### SSI investigation routing (unified — all environments)

Investigation lifecycle routes **always** go through Core, regardless of environment.
Core is the orchestrator: it creates task records, performs dedup, triggers SSI, and tracks status.
This ensures manual (UI) and automated (case-intake) triggers share one code path.

| Operation               | Flow                                                                                                             | Notes                                                                     |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Investigation trigger   | UI → Next.js `/api/ssi/investigate` → Core `POST /investigations/ssi` → SSI Service `POST /trigger/investigate`  | Core creates task, runs dedup, dispatches to SSI                          |
| Status polling          | UI → Next.js `/api/ssi/investigate/{id}` → Core `GET /tasks/{id}`                                                | SSI pushes updates via `TaskStatusReporter`; Core is single status source |
| Evidence download       | UI → Next.js `/api/ssi/report/{id}` → Core `GET /investigations/ssi/{id}/report.pdf`                             | Local: serves from FS; Cloud: 307 → GCS signed URL                       |
| Automated trigger       | Core case-processing → Core `POST /investigations/ssi` → SSI Service                                             | Same endpoint, no UI involvement                                          |

**Local dev requirement:** Core must have `I4G_SSI__SERVICE_URL=http://localhost:8100` so it can reach the SSI service.

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

---

## 7. Common Pitfalls (Do NOT Repeat)

1. **404 on artifact links** — Core returns `/cases/{id}/evidence/{doc_id}`. UI must prefix with `/api` for browser-facing `<a href>`. Without `/api`, the browser hits a nonexistent Next.js page route.

2. **Assuming auth issues locally** — `I4G_ENV=local` bypasses ALL auth. A 404 or 500 is never an auth problem locally. Check: route existence, data existence, correct URL path.

3. **Duplicate PDF URLs** — SSI PDF report has TWO paths that reach the same core endpoint:
   - Dedicated: `/api/ssi/report/{scan_id}` (preferred — handles GCS redirects, sets Content-Disposition)
   - Catch-all: `/api/investigations/ssi/{scan_id}/report.pdf` (also works but less robust)
     Always use the dedicated route for consistency.

4. **SSI investigation routes always go through Core** — Investigation trigger and status polling are always proxied via Core API, not direct to SSI. This applies to both local and cloud. Core orchestrates the workflow (task creation, dedup, dispatch, status tracking). Only eCX routes go direct to SSI. If adding new SSI investigation endpoints, route them through Core.

5. **Router registration order** — SSI routers with specific paths (wallets, evidence) must be registered before the `/{scan_id}` catch-all. Changing order in `app.py` can break routing.

6. **Server vs client component data** — Server components fetch from `I4G_API_URL` directly. Client components fetch from `/api/*` (proxied). URLs returned by the API are meant for the API, not the browser. When rendering links in server components, add the `/api` prefix.

7. **Evidence storage abstraction** — `EvidenceStorage` abstracts local FS vs GCS. In local mode it uses `data/evidence/`. In cloud it uses GCS buckets with signed URLs. Never hardcode file paths — use the `EvidenceStorage` interface.

8. **Case ≠ Review (and “cases” in the UI)** — `case_id` and `review_id` are different. `review_queue` now has a FK to `cases.case_id`. `cases` is the authoritative entity holding classification, description, metadata, and tags. `scam_records` is a write-through search cache only. `get_extended_case()` joins `review_queue` → `cases` (not `scam_records`). Timeline events (`review_actions`) are keyed by `review_id`, not `case_id`. The UI calls them **cases**; the API routes use `/reviews/` — this is an intentional alias. Don't rename one without the other.

9. **TIFAP is NOT a separate service** — The Threat Intelligence & Fraud Analytics Platform reads from core's database via SQLAlchemy directly — no HTTP hop, no separate service, no dedicated Cloud Run service. There is no `/tifap/` API prefix. Campaign intelligence, entity stats, and analytics are served by existing core routers: `/intelligence/`, `/impact/`, `/campaigns/`, and `/exports/`. Do NOT create a TIFAP service or add a proxy route for it — the data access is intentionally in-process for performance.
# Pre-Merge Review Checklist

When the user requests a **pre-merge review**, execute every section below
against **all files changed in the current work stream** (use `git diff` or the
file list from the task context). This is a mandatory, comprehensive pass — not
a quick skim.

---

## 1. Coding Standards (ref: `copilot/.github/shared/general-coding.instructions.md`)

### Type Hints

- Every function/method has **parameter type hints** and a **return type**.
- Use `from __future__ import annotations` where needed for forward refs.
- For expensive runtime-only imports used solely as types (e.g., Playwright
  `Page`), guard with `TYPE_CHECKING`.

### Docstrings & Comments

- **Public** functions, methods, classes, and modules have a Google-style
  docstring with `Args:`, `Returns:`, and `Raises:` sections where applicable.
- **Private** helpers (`_name`) have at least a one-line docstring explaining
  purpose.
- Non-obvious logic has inline comments explaining _why_, not _what_.

### Naming

- Python: `snake_case` functions/variables, `PascalCase` classes, `UPPER_SNAKE_CASE`
  constants.
- TypeScript: `camelCase` variables/functions, `PascalCase` components/types,
  `UPPER_SNAKE_CASE` constants.
- Terraform: `snake_case` everything.

### Formatting

- Python: Black-compatible at 120 chars, isort (profile = black).
- TypeScript: `pnpm format` (Prettier).
- Terraform: `terraform fmt`.

### Imports

- No unused imports.
- Absolute imports preferred in Python. No wildcard imports.

---

## 2. Code Quality

- **No dead code.** Remove commented-out blocks, unused variables, unreachable
  branches.
- **No bare `except:`.** Catch specific exceptions.
- **No hard-coded secrets, paths, or URLs.** Use settings / env vars.
- **Error handling is present** where I/O or external calls occur (try/except
  with logging).
- **Variable scoping is safe** — no variables referenced outside the block where
  they are assigned (e.g., conditional assignments that might not execute).

---

## 3. Architecture & Patterns

- New code follows existing patterns in the codebase (store factories, settings
  access, audit logging).
- No circular imports.
- Database changes have an Alembic migration with a descriptive message and
  idempotent guards.
- Pydantic models use `snake_case` internally with `alias_generator` for wire
  format — no manual casing translation.

---

## 4. Tests

- Changed/new logic has corresponding unit tests.
- Run the full test suite (`pytest tests/unit`) and confirm **zero failures**.
- Note any intentionally skipped test files and the reason.

---

## 4b. Quality Gates (repo-specific — mandatory)

Each repo has its own quality gate. **Always run from that repo's root directory
using the correct environment.**

### Python repos (pre-commit double-pass)

| Repo    | Conda env | Command                                           |
| ------- | --------- | ------------------------------------------------- |
| `core/` | `i4g`     | `conda run -n i4g pre-commit run --all-files`     |
| `ssi/`  | `i4g-ssi` | `conda run -n i4g-ssi pre-commit run --all-files` |

**Procedure:**

```bash
cd <repo-root>                            # must be the repo containing .pre-commit-config.yaml
conda run -n <env> pre-commit run --all-files   # Pass 1 — formatter hooks auto-fix files
git add -u                                # stage auto-fixed files (black, isort, ruff --fix)
conda run -n <env> pre-commit run --all-files   # Pass 2 — must exit clean, zero files modified
```

### UI repo

The UI repo has no pre-commit hooks. Run the following gates in order:

```bash
cd ui/
make check       # pnpm format && pnpm lint && pnpm test (unit)
make test-smoke  # pnpm --filter web test:smoke
make build       # pnpm build (Next.js production build)
```

All steps must pass with zero errors. If `pnpm format` modifies files,
stage them and re-run `make check` to confirm a clean pass.
`make build` catches type errors and import issues that unit tests miss.

- Pass 1 routinely modifies files (Black reformats, isort reorders, ruff auto-fixes). This is expected.
- After Pass 1, **always** `git add -u` the auto-fixed files before Pass 2.
- Pass 2 must show every hook as **Passed** with no files modified. If any hook still fails, fix the root cause — do not re-run in a loop.
- The pre-merge review is **not complete** until a clean Pass 2 is confirmed for every repo that has changes.

---

## 5. Docs & Config

- If behavior or environment variables changed, update:
  - `planning/change_log.md`
  - `docs/config/` (settings manifest / env-var table)
  - Inline docstrings / module docstrings
- Markdown follows present-tense, active-voice, second-person style.

---

## 6. Reporting

After the review, produce a summary:

1. **Issues found** — list each with file, line, and category (e.g., "missing
   docstring", "unused import", "unsafe variable scope").
2. **Fixes applied** — list what was corrected.
3. **Test results** — pass/fail count and any skipped suites.
4. **Remaining items** — anything that could not be auto-fixed and needs human
   decision.
---
applyTo: "**"
---

# Documentation Governance

Every engineer on the I4G platform is a documentation owner. These rules apply to all repos in the i4g workspace.

## Definition of Done for Documentation

A PR is **not ready to merge** until documentation is current. The reviewer is responsible for enforcing this — not CI.

**For any PR that adds, changes, or removes component behavior:**

- [ ] **Tier 2 TDD updated** — the affected subsystem's design document reflects the change. If the subsystem has no TDD, create a stub using [`tdd-template.md`](tdd-template.md).
- [ ] **Config manifest updated** — if any `I4G_*` or `SSI_*` env vars were added or removed, update `core/docs/config/settings_manifest.yaml` and the relevant `settings.default.toml`.
- [ ] **Integration contracts updated** — if the change affects a cross-service boundary (UI↔Core, Core↔SSI, SSI→Core, Scheduler→Job), update [`planning/architecture/integration_contracts.md`](../../../planning/architecture/integration_contracts.md).
- [ ] **End-user doc updated** — if the change affects a workflow visible in the analyst console or API reference, update the relevant page in `docs/book/`.
- [ ] **Change log entry added** — add a bullet point to `planning/change_log.md` under today's date section.

**For a PR that only touches infrastructure (Terraform):**

- [ ] If a new Cloud Run service or job is added: update `planning/architecture/system_narrative.md` Section 2 (Component Inventory).
- [ ] If a new Cloud Scheduler job is added: update `infra/docs/scheduler_inventory.md`.
- [ ] Add a change log entry.

## Ownership Model

Documentation ownership follows code ownership. The engineer who writes the code owns the doc update for that changeset. The tech lead for the affected component is the documentation reviewer.

| Tier | Reviewer |
|---|---|
| 0 — System narrative | Chief Architect reviews any substantive change |
| 1 — System architecture | CTO or Chief Architect |
| 2 — Component TDD | Repo tech lead |
| 3 — Operational guides | Repo tech lead or SRE lead |
| 4 — End-user docs | CPO (editorial), tech lead (accuracy) |
| 5 — Copilot intelligence | CTO |

## Staleness Detection

Every Tier 1 and Tier 2 document must include a `Last Verified` date in its front matter or header:

```markdown
**Last Verified:** [Month YYYY]
```

**A document is stale if its `Last Verified` date is more than 90 days ago.**

Use the staleness banner for sections you cannot confidently verify without reading the code:

```markdown
> ⚠️ **Verification needed** — This section was last checked [date].
> If you find it out of date, update it and remove this banner.
```

Do not remove the banner without actually verifying the content against the running code.

## Quarterly Documentation Review

Once per quarter, each tech lead performs a documentation review for their area:

1. Open `planning/architecture/doc_audit_matrix.md`.
2. For each Tier 1–2 document with a `Last Verified` date older than 90 days: verify it against the current code and update both the document and the audit matrix.
3. For any `GAP` entries in your area: either create the document or log a specific blocker.
4. Update `Last Verified` dates in the audit matrix.
5. Add a change log entry summarizing what was reviewed.

**Target:** Zero Tier 1–2 documents with `Last Verified` older than 90 days by the end of each quarterly review.

## New Component Checklist

When adding a new component (service, job, major subsystem):

1. Create `<component>_tdd.md` using [`tdd-template.md`](tdd-template.md).
2. Add the component to `planning/architecture/system_narrative.md` Section 2 (Component Inventory) and Section 3 (Integration Map).
3. Add cross-service integration contracts to `planning/architecture/integration_contracts.md`.
4. Add a row to `planning/architecture/doc_audit_matrix.md`.
5. Update the master TDD for the owning repo's `tdd.md` (add a row to the Subsystem Index).
6. If it's a Cloud Run service/job: update `infra/docs/service_catalog.md`.
7. If it has a scheduler: update `infra/docs/scheduler_inventory.md`.
8. If it's user-facing: add a page to `docs/book/`.
---
applyTo: "**/planning/handoffs/**,**/planning/tasks/**"
---

# Task Manifest — Planner→Executor Handoff Contract

This document defines the **Task Manifest** format used when a Planner hands work to an Executor. A manifest is the single source of truth for _what_ gets built; the Executor implements it faithfully and does not re-plan.

## When a manifest is required

- Any task the Planner expects to take more than ~20 minutes of Executor time.
- Any task touching more than one repo.
- Any task involving migrations, new env vars, or public API changes.

Small, single-file, single-repo edits can skip the manifest and run `/work-on-task` directly.

## Scope cap: one phase per manifest

**A manifest must fit in a single Executor session and end in a single commit per touched repo.** If the work naturally decomposes into phases (A, B, C…), issue **one manifest per phase** and chain them via `/verify-handoff` between phases. Do not bundle multiple phases into one artifact.

Heuristics for splitting:

- **> ~8 Executor turns estimated** → split.
- **> 1 repo requires an independently runnable verification block** → split.
- **> 1 "commit after this step" checkpoint** in the step-by-step → split.
- **Files-to-create list > ~8 items** → split.

Violating this cap is the #1 cause of "Executor started but never committed and never ran verification" failures (see `/memories/repo/workflow-patterns.md`).

## Commit + verify discipline

Every manifest MUST include, as the final step-by-step entry:

1. Run every command in `<verification>` and paste the exit code + last ~20 lines of output into the execution report.
2. `git add -A` and commit in each touched repo with a message referencing the manifest filename.
3. Stop. Do not begin any adjacent work.

An Executor session that ends with uncommitted files or unrun verification commands is a **failed** handoff regardless of how much code was written.

## File location

- `planning/handoffs/<YYYY-MM-DD>-<slug>.manifest.md` (one file per handoff), OR
- Embedded as a fenced block in an existing `planning/tasks/*.md` sprint plan.

## Required structure

Use Markdown for readability **with XML tags around contract-critical sections**. Executors are instructed to treat XML-tagged content as authoritative; surrounding Markdown is context.

````markdown
# <Task title>

<contract>
**Role:** Executor
**Planner model:** <e.g., Claude Opus 4.7>
**Manifest version:** 1
**Estimated scope:** <S | M | L>
**Repos touched:** core, ui
</contract>

## Goal

<One or two sentences. High-level "done looks like…".>

## Context

<Background the Executor needs: linked PRD, relevant architecture notes, upstream decisions.
Link to `copilot/.github/shared/architecture-cheatsheet.instructions.md` sections rather than re-stating.>

<files>
### Files to modify

- `core/src/i4g/api/review.py` — add `POST /reviews/{id}/reopen` endpoint; reuse `ReviewStore.reopen()`.
- `core/tests/unit/api/test_review.py` — add coverage for the new endpoint (success + 404).
- `ui/apps/web/...` — wire a "Reopen" button in the review detail page.

### Files to create

- `core/src/i4g/api/schemas/reopen.py` — Pydantic request/response models.

### Files NOT to touch

- `core/src/i4g/worker/**` — out of scope for this task.
  </files>

## Step-by-step

1. …
2. …
3. …

<do_not>

- Do not change the database schema.
- Do not introduce new dependencies.
- Do not refactor adjacent code, even if it looks wrong — log it in `planning/change_log.md` as a follow-up instead.
- Do not skip tests.
  </do_not>

<verification>
### Acceptance criteria

- [ ] `conda run -n i4g pytest tests/unit/api/test_review.py -x` passes.
- [ ] `conda run -n i4g pre-commit run --files <changed files>` clean on pass 2.
- [ ] Manual: `curl -X POST .../reviews/<id>/reopen` returns 200 with expected JSON.
- [ ] UI: Reopen button appears only for closed reviews; click triggers success toast.

### Commands to run

```bash
conda run -n i4g pytest tests/unit/api/test_review.py -x
cd ui && pnpm test -- review-detail
```
````

</verification>

## If blocked

Run `/clarify` — produce a short structured question and stop. Do not guess.

```

## Rules for the Executor

1. **Do not re-plan.** The manifest is the plan. If it's wrong, stop and `/clarify`.
2. **Treat `<contract>`, `<files>`, `<do_not>`, and `<verification>` as authoritative.** If other prose conflicts with them, the XML tags win.
3. **Stay inside the "Files to modify / create" list.** If a file outside that list needs a change, stop and `/clarify`.
4. **Every acceptance criterion must be executed and reported** in the final summary (pass/fail, with the command output).
5. **Prefer minimal change.** No refactors, no stylistic edits outside touched lines.

## Rules for the Planner

1. **Write verification as commands**, not prose. Executors follow commands more reliably than narratives.
2. **Use `<do_not>` liberally** to prevent drift — models are more obedient about explicit negatives than implicit scope.
3. **Link, don't repeat** — reference `architecture-cheatsheet.instructions.md` and `general-coding.instructions.md` by path; do not inline their content.
4. **Version the manifest** if you revise it mid-execution. Bump `Manifest version` and tell the Executor to restart from the affected step.
5. **One phase per manifest.** If the work needs phase checkpoints, it needs multiple manifests. See "Scope cap" above.
6. **Hoist shared primitives first.** If a pattern (e.g., `ProviderGate`, `SkippedResult`, a shared client base) will be reused by ≥2 modules in this or a future manifest, create its home file FIRST in the step order — do not let the Executor inline it into the first consumer.
7. **Predecessor gates check commit existence, not push state.** Write Step 1 as: "Confirm commit matching `<SHA or message prefix>` appears in `git log --oneline` of `<repo>`." Do NOT gate on `origin/main` unless this manifest's verification literally consumes a CI-built artifact (e.g., a registry image rebuilt on push). If a CI artifact IS required, write that as a SECOND, separate gate ("artifact `<X>` updated since `<predecessor commit timestamp>`") so failure modes are distinguishable.
8. **Split manifests at the local/external boundary.** If part of `<verification>` is local-deterministic (file edits, fmt, lint, unit tests, `terraform validate`) and part requires external infrastructure (`terraform plan`/`apply`, GCP/AWS API calls, cluster smoke, network egress), produce TWO manifests. The local half lands immediately and is durable; the external half files separately and triggers when the dependency is back. Bundling them produces unblockable manifests when the external dep is unavailable (billing disabled, network outage, missing creds).
9. **Default `<do_not>` additions for every manifest:**
   - "Do not delete or move files outside this manifest's `<files>` list, including prior sprint manifests."
   - "Do not re-save sections of a settings/config file you are not extending. Verify with `git diff -w` that untouched blocks show no whitespace-only diffs."
   - "Do not leave uncommitted files at end of session. If verification fails, commit a WIP with an explicit `[WIP]` prefix and stop for `/clarify`."

```
---
applyTo: "**/osint/**,**/ingestion/phishdestroy/**,**/worker/jobs/merklemap_*.py,**/worker/jobs/phishdestroy_*.py,**/review.py,**/discoveries/**,**/actors/**"
---

# PhishDestroy Provider Gating (quota-gated skip contract)

The PhishDestroy integration depends on several paid third-party APIs whose budgets are
**not guaranteed at implementation time**. This file defines the single pattern every
ingestion job, OSINT module, API route, and UI surface must follow so that:

1. Code builds and tests pass regardless of key availability.
2. Disabled providers **never** surface as silent failures — they surface as explicit
   "skipped because quota-gated" signals at every layer (logs, audit log, review UI).
3. Flipping a provider from disabled → enabled is a single config change plus a Secret
   Manager entry — no code changes, no redeploy of unrelated services.

## 1. Gated providers (as of Sprint 0)

| Provider | Status | Keyed off | Funded? | Consumes |
| --- | --- | --- | --- | --- |
| `merklemap` | funded (dev key available) | `providers.merklemap.enabled` + `api_key` | yes | Sprint 1 merklemap tail |
| `whoxy` | **budget TBD** | `providers.whoxy.enabled` + `api_key` | pending | Sprint 3 `whoxy_reverse.py` |
| `ghunt` | ops-readiness TBD | `providers.ghunt.enabled` + cookie blob | pending | Sprint 3 `ghunt.py` |
| `virustotal` | funded | `providers.virustotal.enabled` + `api_key` | yes | existing SSI |
| `urlscan` | funded | `providers.urlscan.enabled` + `api_key` | yes | existing SSI |
| `securitytrails` | budget TBD | `enrichment.securitytrails_api_key` | pending | existing enrichment |

When a provider's status is not "funded", the PRD acceptance metric(s) that depend on it
(§10) are considered **deferred**, not failing. See §6.

## 2. Settings shape

Provider config lives under a `[providers.<name>]` section in the repo's settings TOML.
Both `core/` and `ssi/` expose identical schema; env-var aliases are `I4G_PROVIDERS__*`
and `SSI_PROVIDERS__*` respectively.

```toml
[providers.merklemap]
enabled = false              # flip to true after key is set
api_key = ""                 # injected via Secret Manager in dev/prod; paste locally in settings.local.toml
base_url = "https://api.merklemap.com"
rate_limit_per_sec = 0       # 0 = no client-side limit; provider limit governs

[providers.whoxy]
enabled = false
api_key = ""
monthly_query_cap = 0        # 0 = no cap; set to budget guardrail once funded

[providers.ghunt]
enabled = false
cookie_blob_path = ""        # path to JSON cookie dump; injected via Secret Manager in cloud
```

Adding a new gated provider requires an entry here **and** a row in §1.

## 3. Runtime contract: `ProviderGate`

Every gated module must gate at the public entry point, not inside helpers. The shape:

```python
from i4g.providers import ProviderGate, SkippedResult  # naming is illustrative

GATE = ProviderGate("whoxy")

async def scan(target: str) -> ScanResult | SkippedResult:
    if not GATE.enabled:
        return GATE.skip(reason="quota_gated", detail="whoxy disabled in settings")
    ...
```

Rules:

- `ProviderGate(name).enabled` returns `True` only if BOTH `providers.<name>.enabled` is
  true AND the required credential (api_key / cookie blob) is non-empty.
- `SkippedResult` is a structured value, never an exception. It carries
  `reason ∈ {"quota_gated", "auth_expired", "rate_limited", "disabled"}` plus a free-form
  `detail` string.
- Orchestrators and ingestion jobs must treat `SkippedResult` as a first-class outcome:
  count it in metrics (`skipped_total{provider=..., reason=...}`) and write one
  `audit_log.log_action(action="provider_skipped", ...)` per skip.
- Never return a default/empty payload when gated. Callers must be able to distinguish
  "provider ran and found nothing" from "provider did not run".

## 4. Surfacing at boundaries

- **Review UI / Discoveries UI:** show a "provider skipped" badge on any scan result that
  includes `skipped[]`. Click reveals provider name + reason + last-enabled date.
- **API responses:** the outer `ScanResult` / `ReviewRecord` schema has a `skipped` array
  of `{provider, reason, detail}`. Clients already ignore unknown fields; adding the
  array is non-breaking.
- **Dossier rendering:** insert a "Coverage gaps" section listing skipped providers with
  reason. Dossier signing includes the skipped list so gaps are auditable after the fact.
- **Metrics (Sprint 4 SLOs):** `provider_skipped_total{provider,reason}` becomes a
  dashboard panel. Persistent non-zero `quota_gated` is informational; persistent
  `auth_expired` is alertable.

## 5. Terraform / deploy gating

Infra changes for a gated provider must ship **as disabled**:

- `enable_whoxy` etc. default to `false` in tfvars.
- The Cloud Run job / Scheduler is created (so enabling is a var-flip) but suspended or
  set to `min_instances = 0` when disabled.
- Secret Manager entries are created with an empty payload; funding the key is a pure
  secret-version bump, no Terraform apply.

This is also why the Sprint 1 merklemap tail ships **enabled** (we have a dev key) while
the Sprint 3 whoxy/ghunt modules ship **disabled** with the code paths in place.

## 6. PRD acceptance-metric deferral

When a gated provider is unfunded, affected PRD §10 acceptance metrics move to a
"deferred" column in the sprint verification report. They are NOT marked failing and do
NOT block merge. The report must list:

- Which metric is deferred.
- Which provider gates it.
- What budget or ops-readiness decision unblocks it.

This keeps the merge/verify loop unblocked on things the engineering team does not
control (budget, counsel, upstream availability).

## 7. Rotation hooks

Rotation is a three-step sequence, identical for every provider:

1. Revoke or re-issue the key at the provider's console.
2. Paste new value into `<repo>/config/settings.local.toml` under
   `[providers.<name>] api_key = "..."` **or** push a new Secret Manager version (dev
   and prod use Secret Manager; local dev uses the TOML override).
3. Restart the consuming service (`uvicorn` reload locally; Cloud Run rolls on next
   revision).

Runbook bodies for each provider live in `copilot/docs/runbooks/` (Sprint 4 deliverable).
---
applyTo: "**/ingestion/phishdestroy/**,**/worker/jobs/phishdestroy_*.py,**/worker/jobs/merklemap_*.py,**/osint/blocklist_aggregator.py,**/osint/merklemap_client.py,**/osint/ctlog_lookup.py,**/alembic/versions/phishdestroy_*.py"
---

# PhishDestroy Provenance Contract (frozen Sprint 0)

Every row written by any PhishDestroy ingestion path — `destroylist`, `ScamIntelLogs`
archive, `DestroyScammers` actor data, blocklist aggregator, merklemap tail — **MUST**
carry the provenance block defined here. No exceptions.

This file is authoritative. If the PRD or a task manifest conflicts with it, this file
wins; update the PRD, not the data.

## 1. `source_provenance` JSON shape

Every `source_provenance` JSON column stores an object with exactly these fields:

```json
{
  "source": "phishdestroy.destroylist",
  "team": "TrustWalletPanel",
  "commit_sha": "83d0307420fcc865fcb8a34b8c454acbc6d56f1f",
  "record_id": "domains.txt#L412",
  "ingested_at": "2026-04-23T14:17:03Z",
  "ingest_job": "i4g-jobs-ingest-phishdestroy-archive",
  "ingest_job_run_id": "cloudrun-exec-abc123"
}
```

### Field rules

| Field | Required | Type | Rule |
| --- | --- | --- | --- |
| `source` | yes | string | Dotted identifier, one of the values in §3. Never free text. |
| `team` | conditional | string | Required for archive / actor data; omit for feeds not scoped to a team (e.g. merklemap, blocklist aggregator). |
| `commit_sha` | yes | string | Full 40-char hex SHA of the upstream repo HEAD at ingest time. Pinned SHAs in §4. For live feeds (merklemap), use the `commit_sha` of the deployed **filter config**, not the feed. |
| `record_id` | yes | string | Stable pointer to the exact source record. See §2. |
| `ingested_at` | yes | string | RFC 3339 / ISO 8601 UTC with `Z` suffix. |
| `ingest_job` | yes | string | Cloud Run job resource name, or CLI command name for local runs. |
| `ingest_job_run_id` | no | string | Cloud Run execution ID when available. |

No extra keys. If you need more context, promote it to a first-class column.

## 2. `record_id` rules

`record_id` must be **deterministic and reproducible** from the upstream source. If two
runs against the same `commit_sha` produce different `record_id` values for the same
logical row, the rule is wrong and must be fixed before merge.

| Source | `record_id` format |
| --- | --- |
| `phishdestroy.destroylist` | `sha256(normalized_indicator)` (hex) when sourced from `DestroyScammers/data/data.json`; `<path>#L<line>` when sourced from the standalone repo (not currently checked out). |
| `phishdestroy.archive.iocs` | `<team>/iocs.json#<jsonpointer>` (RFC 6901) |
| `phishdestroy.archive.chat` | `<team>/chat/<filename>#<message_index>` |
| `phishdestroy.archive.damage` | `<team>/successful_thefts/<filename>#<jsonpointer>` |
| `phishdestroy.archive.financial_damage` | `<team>/successful_thefts/result.json#<message_id>` |
| `phishdestroy.actors` | `data.json#/<actor_key>` |
| `phishdestroy.registrants` | `registrants.json#/<pivot_type>/<pivot_value>` |
| `blocklist.<source>` | `sha256(normalized_indicator \| source)` (hex) |
| `merklemap.tail` | `sha256(domain \| first_seen_unix)` (hex) |
| `ctlog.crtsh` | `crtsh:<entry_id>` |

## 3. Allowed `source` values

Controlled vocabulary. Reject any value outside this list at write time.

```
phishdestroy.destroylist
phishdestroy.archive.iocs
phishdestroy.archive.chat
phishdestroy.archive.damage
phishdestroy.archive.financial_damage
phishdestroy.archive.infrastructure
phishdestroy.archive.brands
phishdestroy.actors
phishdestroy.registrants
blocklist.metamask
blocklist.scamsniffer
blocklist.openphish
blocklist.seal
blocklist.enkrypt
blocklist.phishdestroy
blocklist.polkadot
blocklist.cryptofirewall
merklemap.tail
ctlog.crtsh
```

Adding a new `source` requires editing this file and the provenance vocabulary test
in `core/tests/unit/ingestion/test_provenance.py`.

## 4. Pinned upstream commit SHAs (Sprints 1–2 baseline)

These SHAs are the baseline for Sprint 1 / Sprint 2 ingestion. When re-ingesting from a
newer upstream, bump the pin here, update `change_log.md`, and re-run the ingest job —
idempotency (§5) will update rows in place.

| Upstream repo | Local checkout | Pinned SHA | Pinned date |
| --- | --- | --- | --- |
| `github.com/phishdestroy/ScamIntelLogs` | `phishdestroy/ScamIntelLogs` | `83d0307420fcc865fcb8a34b8c454acbc6d56f1f` | 2026-03-01 |
| `github.com/phishdestroy/DestroyScammers` | `phishdestroy/DestroyScammers` | `c40cbbf527dd9e5e232090346e1a8ceab32d1683` | 2025-11-30 |
| `github.com/phishdestroy/destroylist` | `phishdestroy/DestroyScammers/data/data.json` (`registrants.json`) | see DestroyScammers SHA | — |
| `github.com/phishdestroy/merklemap-cli` | `phishdestroy/merklemap-cli` | `550cb04aa633c000724c339ada085c59444d5b78` | 2024-10-06 |

> The standalone `destroylist` repo is not checked out locally; the same domain list is
> reachable through `DestroyScammers/data/data.json` and is covered by that SHA. If the
> Sprint 1 `destroylist` ingestion job pulls from the standalone repo instead, pin its
> SHA here in the same update.

## 5. Idempotency key

All ingestion jobs MUST be idempotent under this key:

```
(source, team, commit_sha, record_id)
```

- `team` contributes to the key only when the `source` entry in §3 includes a team scope.
- The composite key is the natural UNIQUE index for staging tables; production tables
  index it as a non-unique composite because the same logical row can be emitted by
  multiple ingest runs (newer `commit_sha`) and upsert on match.
- Re-running a job with the **same** `commit_sha` must be a no-op (unchanged rows).
- Re-running with a **newer** `commit_sha` updates in place and appends the new SHA to a
  `provenance_history` JSON array on the row (schema detail handled in Sprint 1
  migration).

## 6. Sensitive-column marker

Columns containing PII per PRD §11 Q3 (real names, chat transcripts, leak passwords,
raw contact addresses) MUST be flagged with `info={"sensitive": True}` at the SQLAlchemy
column definition. Example:

```python
real_name = Column(String, info={"sensitive": True})
```

Downstream rules that key off this marker (automatically enforced by tests):

- Dossier renderer redacts unless reader has `role=senior_analyst`.
- Every read of a `sensitive` column emits an `audit_log.log_action` with actor identity
  + `reason_code` (required — API rejects requests without one).
- BigQuery export policy: sensitive columns land in a separate, access-restricted view.

A unit test (`tests/unit/schema/test_sensitive_markers.py`) asserts the expected set of
sensitive columns; changing the set requires updating the test in the same commit.

## 7. What this contract does NOT cover

- Storage layout of evidence blobs (chat exports, screenshots) — see Sprint 2.4 and
  `core/docs/design/storage.md`.
- Access-control mechanics (RBAC, reason-code enforcement) — see Sprint 3.5 and the
  API instructions.
- Rate-limit / quota budgeting for paid providers — see
  `phishdestroy-provider-gating.instructions.md`.
