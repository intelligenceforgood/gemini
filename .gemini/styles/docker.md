# Docker Standards

This document outlines the standard Dockerfile patterns for the I4G workspace, particularly for Cloud Run and Vertex AI components.

## Base Images

- Use specific, slim base images (e.g., `python:3.11-slim` for Core/SSI Python services, `node:18-alpine` or `node:20-alpine` for the UI).
- Avoid using the `latest` tag; always pin to a specific major/minor version to ensure reproducible builds.

## Multi-Stage Builds

To keep the final image size minimal and reduce the security attack surface, all Dockerfiles must leverage multi-stage builds.

- **Builder Stage**: Compile dependencies, build assets, and prepare the environment. For Python, this typically involves installing packages via `pip` or `conda` into a virtual environment.
- **Runtime Stage**: Copy only the essential compiled artifacts and dependencies from the builder stage. Source code should be copied over at the end.

## User Permissions (Least Privilege)

- **Never run as root**. Create a dedicated non-root user (e.g., `i4g` or `appuser`) and switch to it using the `USER` directive.
- Ensure you set appropriate ownership on working directories during the build process _before_ switching users.

## Layer Caching

- Order `COPY` and `RUN` commands to maximize Docker layer caching.
- Copy `requirements.txt`, `package.json`, or environment files and install dependencies **before** copying the rest of the application code.

## Entrypoints & Arguments

- Keep entrypoints clean and use the JSON array syntax (e.g., `ENTRYPOINT ["python", "-m", "module"]`).
- Images should be flexible enough to support overriding arguments for sub-commands. For example, the `ssi-svc` image can be reused with `["ecx", "poll"]` arguments for Cloud Run Jobs.

## Environment Variables & Secrets

- Define default, non-sensitive environment variables using `ENV` (e.g., `ENV I4G_ENV=production`).
- **Never bake secrets into the image.** Pass sensitive secrets (such as `SSI_ECX__API_KEY` or `I4G_CRYPTO__PII_KEY`) securely at runtime via Google Cloud Secret Manager.
