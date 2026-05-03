# CI/CD Pipeline Standards

This document outlines the standard CI/CD practices across all I4G platform projects, leveraging GitHub Actions and Google Cloud.

## Pipeline Structure

- **Workflows per Component**: Workflows are localized within component directories (e.g., `ui/.github/workflows`, `core/.github/workflows`, `infra/.github/workflows`).
- **Trigger Paths**: Pipelines should be configured to trigger only when files relevant to that specific component are changed using `paths` and `paths-ignore` filters.
- **Environment Separation**: Maintain strict logical separation between environments (e.g., `dev`, `staging`, `prod`) in deployment steps.

## Job Conventions

- **Naming**: Use clear, action-oriented job names (e.g., `build-and-test`, `deploy-to-cloud-run`).
- **Dependencies**: Use the `needs` keyword to enforce job sequencing. Deployment jobs must always wait for the successful completion of linting, testing, and security scanning jobs.
- **Caching**:
  - For Python (`core/`, `ssi/`, `ml/`), cache dependency environments (e.g., `conda` or `pip`).
  - For Node.js (`ui/`, `mobile/`), cache the `pnpm store` to accelerate dependency installation.

## Deployment Workflows

- **Docker Image Builds**: Build deployment artifacts as Docker images following the workspace multi-stage Docker standards, and push them to Google Artifact Registry.
- **Cloud Run & Vertex AI**: Deploy updated images to Google Cloud using standard actions (e.g., `google-github-actions/deploy-cloudrun`) or custom Terraform executions via `infra/`.
- **Authentication & Secrets**:
  - **Never hardcode secrets or service account keys.**
  - Use Workload Identity Federation (WIF) to authenticate GitHub Actions to Google Cloud securely.
  - Inject necessary runtime secrets via Google Cloud Secret Manager.

## Quality Gates

- **Formatting & Linting**: All PR pipelines must enforce code formatting (e.g., Black, Ruff, Isort for Python; Prettier for UI).
- **Test Coverage**: Automated tests (e.g., `pytest` for Python, `turbo test` for UI) must run and pass on every pull request before merging is allowed.
- **Database Migrations**: Changes to `core/src/i4g/migrations/` must be validated in an isolated CI database to ensure no conflicting heads or destructive changes occur.
