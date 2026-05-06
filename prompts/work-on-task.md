---
agent: agent
description: "Execute a task, or tasks in the same sprint (or phase) with proper testing and documentation."
---

# Work on Task

Execute a single implementation task, or a group of tasks in the same sprint (or phase) with proper testing and documentation.

## Steps

1. **Understand the task.** Scan the workspace and read the relevant code to understand what exists. Don't modify code you haven't read. You are encouraged to scan the entire folder if needed to gain full context.

2. **Implement.** Write the code following the standards for the file type. Key principles:
   - Settings access via `get_settings()`, not hard-coded values
   - Stores via factories in `src/i4g/services/factories.py`
   - Type hints on every function, Google-style docstrings on public methods
   - Specific exception handling (no bare `except:`)
   - Ensure the new code follows the rules defined in the relevant style guides (e.g., Python, TypeScript/React).

3. **Test.** Write or update tests for the changed logic:
   - Unit tests under `tests/unit/`
   - Run: `conda run -n i4g pytest tests/unit -x` (stop on first failure)
   - If adding env vars, add coverage under `tests/unit/settings/`
   - Identify and run any relevant verification commands to ensure the change works as intended.

4. **Document.** If behavior changed:
   - Update `docs/` if user-facing
   - Update config manifests if env vars changed
   - Note in `planning/change_log.md`
   - **CRITICAL:** If working from a task plan or manifest, explicitly update the document to check off the completed tasks (e.g. change `[ ]` to `[x]`). If for some reason the task isn't testable yet (lack of API key, or GCP project isn't ready), mark it as `[-]`.

5. **Validate locally.** Run pre-commit hooks to catch formatting issues early:

   ```bash
   conda run -n i4g pre-commit run --files <changed-files>
   ```

6. **Summarize.** Report what was done, tests that pass, and any follow-up items. Keep explanations lean and ensure no conversational fluff is introduced. **CRITICAL: Do NOT output the modified code or code diffs in your chat response. Since the files are already updated, simply list the files that were modified.**
