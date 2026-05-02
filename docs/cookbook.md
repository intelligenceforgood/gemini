# GCA Cookbook: Real-World Workflows

This cookbook provides practical examples of how to chain GCA routines together to accomplish complex tasks within the I4G environment.

> ⚠️ **CRITICAL: Agent Mode & Quota Management**
>
> Your daily quota is finite. To avoid running out:
> * **Turn Agent Mode OFF** for planning, drafting PRDs, code reviews of single files, and analyzing logs.
> * **Turn Agent Mode ON** *only* when you need GCA to autonomously edit multiple files, run tests in the terminal, or perform complex, scoped refactoring.
> * Always use explicit tags (`@file:path`, `@folder:path`) to restrict context.

## 🥘 Recipe 1: End-to-End Feature Development

**Scenario**: You need to implement a new API endpoint in the `core` repository and a corresponding frontend component in the `ui` repository.

**Steps**:

1.  **Ideation**: Open GCA Chat and type the `gca-prd` snippet.
    - _Input_: "We need a new endpoint `/api/v1/health/detailed` that returns database connection status, and a React component to display this status on the admin dashboard."
    - _Action_: Save GCA's response as `docs/prd_detailed_health.md`.
2.  **Architecture**: Type the `gca-arch` snippet.
    - _Input_: "Based on `docs/prd_detailed_health.md`, propose the architecture respecting the rules in `.gemini/styles/`."
    - _Action_: Save as `docs/arch_detailed_health.md`.
3.  **Planning (Broad Context)**: Type the `gca-impl` snippet. (Agent Mode: Optional)
    - _Input_: "Create a step-by-step implementation plan for the detailed health feature based on the PRD and Architecture docs. Focus your context on `@folder:docs` and `@folder:planning`."
    - _Action_: Save as `planning/impl_detailed_health.md`.
4.  **Execution (Scoped Context)**:
    - _Rule of Thumb_: ALWAYS scope your execution to specific files to preserve daily token quota.
    - For the API: Ensure **Agent Mode is ON**. Type `gca-work` and ask: "Execute Task 1 from `planning/impl_detailed_health.md`. Restrict your context strictly to `@file:core/src/i4g/api/health.py` and `@file:.gemini/styles/python.md`."
    - For the UI: Ensure **Agent Mode is ON**. Type `gca-work` and ask: "Execute Task 2 from `planning/impl_detailed_health.md`. Read only `@file:ui/apps/web/src/components/Health.tsx` and `@file:.gemini/styles/typescript.md`."
5.  **Validation (Agent Mode OFF)**: Type the `gca-review` snippet.
    - _Input_: "Review my uncommitted changes for this feature. Verify against `@folder:core/.gemini/styles/`."
    - _Note_: Turning Agent Mode off here prevents GCA from autonomously scanning the workspace to "find" uncommitted changes, saving massive amounts of tokens. Just use the active source control diff.

## 🐛 Recipe 2: Bug Hunting and Fixing

**Scenario**: A production bug is reported: "User profile updates are failing intermittently with a 500 error."

**Steps**:

1.  **Diagnosis (Agent Mode OFF)**: Paste the stack trace or log snippet into GCA Chat. Type the `gca-log` snippet.
    - _Input_: "Analyze this stack trace: [PASTE LOGS HERE]"
    - _Expected Output_: GCA analyzes the logs and points to the likely failing function or database query.
2.  **Reproduction & Fix (Agent Mode ON)**: Type the `gca-fix` snippet.
    - _Input_: "The `update_profile` function in `@file:core/users/service.py` is failing. Read that file and `@file:core/tests/unit/test_service.py`. Help me write a failing test to reproduce it, then fix the bug."
    - _Expected Output_: GCA provides a unit test, asks you to run it, and then provides the patch to fix the underlying issue.
3.  **Lesson Recording**: Once fixed, type `gca-lesson`.
    - _Input_: "Record the lesson learned regarding database transaction timeouts during profile updates."
    - _Action_: Append the result to a team knowledge base or the `README.md`.

## 🤝 Recipe 3: Context Handoff

**Scenario**: You are ending your day in the middle of a complex refactoring task and need to hand it off to a colleague in another timezone.

**Steps**:

1.  **Create Handoff**: Open GCA Chat and type `gca-handoff`.
    - _Input_: "Create a handoff manifest for the ongoing Auth module refactor. I have completed steps 1 and 2, but step 3 is failing unit tests."
    - _Action_: Save the output to `planning/handoff_auth_refactor.md` and commit/push it.
2.  **Receive Handoff (Colleague)**: Your colleague pulls the branch. They open GCA Chat and type `gca-resume`.
    - _Input_: "Read `planning/handoff_auth_refactor.md` and restore your context so we can continue."
    - _Next Step_: Colleague types `gca-exec` -> "Execute the next pending task in the handoff document."

## 🛠️ Recipe 4: Deep Refactoring with Context

**Scenario**: You need to update a core utility function that is used in 50 different places across the repository.

**Steps**:

1.  **Analyze Impact**: Ask GCA: "I need to change the signature of `format_date()` in `utils/date.ts`. Find all usages across the workspace and tell me the impact."
2.  **Plan Refactor**: Type `gca-plan`.
    - _Input_: "Plan the refactoring steps to safely update all 50 usages of `format_date()`, prioritizing minimizing test breakages."
3.  **Iterative Execution**: Use `gca-work` to execute the refactoring in batches, asking GCA to run tests after each batch to ensure stability.
