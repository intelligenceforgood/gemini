---
applyTo: "*"
---

# Security Standards

This document outlines the standard security practices across all I4G platform projects, ensuring data protection, secure authentication, and least-privilege access.

## 1. Authentication & Authorization

- **Environment-Specific Auth**:
  - **Local (`I4G_ENV=local`)**: All auth is bypassed. `require_token()` returns a mock user (`{"username": "local-dev", "role": "admin"}`). Do not debug 404s/500s as auth issues locally.
  - **Cloud (`I4G_ENV=dev` or `prod`)**: Fronted by Identity-Aware Proxy (IAP). Validates `X-API-KEY` or Google Identity Platform OIDC JWTs.
- **Proxying**: Next.js server-side API routes must inject IAP headers via `getIapHeaders()` before proxying to the core API.
- **Role-Based Access Control (RBAC)**: Enforce roles (e.g., `user`, `analyst`, `admin`, `leo`) for accessing, updating, or deleting cases.

## 2. Input Validation & Data Shapes

- **Backend (Python)**: Use Pydantic models for all request/response validation. Use `CamelModel` with `alias_generator = to_camel` for JSON output. Catch specific exceptions; never use bare `except:`.
- **Frontend (UI)**: Use Zod schemas that strictly match what the FastAPI backend returns. Verify endpoint structures with `curl` before writing new schemas.

## 3. Data Protection & Encryption

- **Encryption at Rest**:
  - **Critical Data** (SSN, bank accounts): Encrypted with AES-256-GCM.
  - **High Sensitivity** (Contact info): Encrypted with Fernet (AES-128-CBC + HMAC-SHA256) at intake. The key (`I4G_CRYPTO__PII_KEY`) is stored in Google Secret Manager.
- **Encryption in Transit**: TLS 1.3 is enforced across all Cloud Run auto-managed services.
- **Data Redaction**: Victim contact info must be redacted from case narrative text during ingestion (replaced with `[VICTIM_EMAIL]`, etc.). Authorized decryption is only available via specific endpoints (e.g., `GET /intakes/{id}/contact`) with strict audit logging.

## 4. Secrets Management

- **No Hardcoded Secrets**: Never commit secrets to source code, `.tfvars`, or bake them into Docker images.
- **Google Secret Manager**: Pass sensitive values (e.g., `I4G_LLM__GEMINI_API_KEY`, `SSI_ECX__API_KEY`) securely at runtime via GCP Secret Manager and Terraform injections.
- **Local Secrets**: Store non-public secrets in `.env.local`, which must be explicitly gitignored.

## 5. Infrastructure & IAM (Least Privilege)

- **Terraform**:
  - Mark sensitive values with `sensitive = true`.
  - Always include the three default `special_group` entries (`projectOwners`, `projectWriters`, `projectReaders`) in BigQuery dataset access blocks.
  - Impersonate service accounts using `gcloud auth application-default login` for state and deployment operations.
- **Docker Containers**: Never run as root. Always create a dedicated non-root user (e.g., `i4g` or `appuser`) and switch to it using the `USER` directive.
- **Workload Identity Federation (WIF)**: Use WIF to securely authenticate GitHub Actions to Google Cloud instead of using long-lived service account keys.
