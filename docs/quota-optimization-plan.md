# GCA Quota Optimization Plan

This plan aims to reduce Gemini Code Assist (GCA) daily quota consumption while maintaining design and implementation consistency across the I4G platform projects.

## Phase 1: Establish Strict Context Scoping (The "Need to Know" Basis)

Shift from an **Implicit Global Context** to an **Explicit Scoped Context**.

1.  **Refine `.gemini/config.yaml` and `.geminiignore`**:
    *   Tighten inclusion/exclusion rules.
    *   Use `.geminiignore` aggressively to ignore test data, binaries, generated files, and third-party libraries.
2.  **Update the Routine Prompts (`prompts/*.md`)**:
    *   Modify execution-focused prompts (`work-on-task.md`, `fix-bug.md`) to explicitly force the use of `@file` or `@folder` tags.
3.  **Phase-Specific Scanning**:
    *   Update `architecture-template.md` and `plan-work.md` to be the *only* routines encouraging broad repository scanning.
    *   The output of these plans *must* explicitly list the target files for the execution phase.

## Phase 2: Mastering Agent Mode Toggling

Establish clear rules for when to toggle Agent Mode in `docs/cookbook.md` and `docs/onboarding.md`:

*   **Agent Mode OFF (High Quota Efficiency):**
    *   General programming questions.
    *   Analyzing log files or stack traces.
    *   Drafting PRDs or initial plans with text context.
    *   Reviewing code for the active file.
*   **Agent Mode ON (Quota Heavy - Use Strategically):**
    *   Executing a specific plan across multiple files.
    *   Running tests and fixing errors.
    *   Refactoring with explicitly scoped `@folder`.

## Phase 3: Eliminating Redundancy

*   Audit the `prompts/` folder to strip out Copilot-specific fluff and ensure prompts are lean and optimized for Gemini.
