# Pre-Merge Code Review

**Instruction:** You are a Principal Engineer. Review the staged changes against `.gemini/styleguide.md` and the original implementation plan.

## 1. Multi-Repo Consistency & Dependencies (CRITICAL)
- **Identify Changed Repos:** Determine which repos across the workspace have staged changes (e.g., `core`, `ssi`, `ui`, `infra`).
- **Cross-Repo Dependencies:** Verify that architectural changes or API updates in one repo are correctly integrated and consumed in dependent repos. Ensure changes are merged simultaneously to keep the system in a clean, consistent state.

## 2. Documentation Synchronization
- Verify that code changes are synchronized with central documentation, specifically `docs/config/settings_manifest.yaml` and `planning/change_log.md`.

## 3. Architecture Consistency
- Does the code follow established patterns (e.g., stores/factories in Python, custom hooks in React)?

## 4. Coding Quality & Quality Gates
- Are there any deviations from the styleguide (type hints, naming conventions, docstrings)?
- **Repo-Specific Gates:** Ensure repo-specific quality gates pass (e.g., Python pre-commit double-pass, UI `make check`/`make build`).

## 5. Test Coverage
- Is this code adequately covered by tests?

## 6. Action Items
List specific bugs, improvements, or "Looks good to me".