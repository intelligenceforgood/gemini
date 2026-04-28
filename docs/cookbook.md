# GCA Cookbook: Real-World Workflows

This cookbook provides practical examples of how to chain GCA routines together to accomplish complex tasks within the I4G environment.

## 🥘 Recipe 1: End-to-End Feature Development

**Scenario**: You need to implement a new API endpoint in the `core` repository and a corresponding frontend component in the `ui` repository.

**Steps**:

1.  **Ideation**: Open GCA Chat and type the `gca-prd` snippet.
    *   *Input*: "We need a new endpoint `/api/v1/health/detailed` that returns database connection status, and a React component to display this status on the admin dashboard."
    *   *Action*: Save GCA's response as `docs/prd_detailed_health.md`.
2.  **Architecture**: Type the `gca-arch` snippet.
    *   *Input*: "Based on `docs/prd_detailed_health.md`, propose the architecture respecting the rules in `.gemini/styleguide.md`."
    *   *Action*: Save as `docs/arch_detailed_health.md`.
3.  **Planning**: Type the `gca-impl` snippet.
    *   *Input*: "Create a step-by-step implementation plan for the detailed health feature based on the PRD and Architecture docs."
    *   *Action*: Save as `planning/impl_detailed_health.md`.
4.  **Execution (Iterative)**:
    *   For the API: Open the `core` repo files. Type `gca-work` and ask: "Execute Task 1 from `planning/impl_detailed_health.md` (Create FastAPI endpoint)."
    *   For the UI: Open the `ui` repo files. Type `gca-work` and ask: "Execute Task 2 from `planning/impl_detailed_health.md` (Create React component)."
5.  **Validation**: Type the `gca-review` snippet.
    *   *Input*: "Review my uncommitted changes across `core` and `ui` repositories for this feature."

## 🐛 Recipe 2: Bug Hunting and Fixing

**Scenario**: A production bug is reported: "User profile updates are failing intermittently with a 500 error."

**Steps**:

1.  **Diagnosis**: Paste the stack trace or log snippet into GCA Chat. Type the `gca-log` snippet.
    *   *Input*: "[PASTE LOGS HERE]"
    *   *Expected Output*: GCA analyzes the logs and points to the likely failing function or database query.
2.  **Reproduction & Fix**: Type the `gca-fix` snippet.
    *   *Input*: "The `update_profile` function in `core/users/service.py` is failing intermittently according to logs. Help me write a failing test to reproduce it, then fix the bug."
    *   *Expected Output*: GCA provides a unit test, asks you to run it, and then provides the patch to fix the underlying issue.
3.  **Lesson Recording**: Once fixed, type `gca-lesson`.
    *   *Input*: "Record the lesson learned regarding database transaction timeouts during profile updates."
    *   *Action*: Append the result to a team knowledge base or the `README.md`.

## 🤝 Recipe 3: Context Handoff

**Scenario**: You are ending your day in the middle of a complex refactoring task and need to hand it off to a colleague in another timezone.

**Steps**:

1.  **Create Handoff**: Open GCA Chat and type `gca-handoff`.
    *   *Input*: "Create a handoff manifest for the ongoing Auth module refactor. I have completed steps 1 and 2, but step 3 is failing unit tests."
    *   *Action*: Save the output to `planning/handoff_auth_refactor.md` and commit/push it.
2.  **Receive Handoff (Colleague)**: Your colleague pulls the branch. They open GCA Chat and type `gca-resume`.
    *   *Input*: "Read `planning/handoff_auth_refactor.md` and restore your context so we can continue."
    *   *Next Step*: Colleague types `gca-exec` -> "Execute the next pending task in the handoff document."

## 🛠️ Recipe 4: Deep Refactoring with Context

**Scenario**: You need to update a core utility function that is used in 50 different places across the repository.

**Steps**:

1.  **Analyze Impact**: Ask GCA: "I need to change the signature of `format_date()` in `utils/date.ts`. Find all usages across the workspace and tell me the impact."
2.  **Plan Refactor**: Type `gca-plan`.
    *   *Input*: "Plan the refactoring steps to safely update all 50 usages of `format_date()`, prioritizing minimizing test breakages."
3.  **Iterative Execution**: Use `gca-work` to execute the refactoring in batches, asking GCA to run tests after each batch to ensure stability.
