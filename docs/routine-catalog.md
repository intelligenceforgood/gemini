# GCA Routine Catalog

This catalog outlines all available Developer Routines provided by the GCA Productivity Framework. These routines are markdown templates located in the `prompts/` directory and are designed to guide GCA through standardized development processes.

## 📋 Planning & Scaffolding

| Routine                 | File                       | Snippet    | Description                                                                                   |
| ----------------------- | -------------------------- | ---------- | --------------------------------------------------------------------------------------------- |
| **PRD Generation**      | `prd-template.md`          | `gca-prd`  | Translates user feedback or feature requests into a structured Product Requirements Document. |
| **Architecture Design** | `architecture-template.md` | `gca-arch` | Drafts a technical approach and architecture design for a given feature.                      |
| **Implementation Plan** | `impl-plan.md`             | `gca-impl` | Breaks down PRD and Architecture documents into actionable, step-by-step developer tasks.     |
| **TDD Scaffolding**     | `tdd-template.md`          | `gca-tdd`  | Scaffolds test files and stubbed implementations based on an implementation step.             |

## 💻 Core Daily Routines

| Routine          | File              | Snippet       | Description                                                                       |
| ---------------- | ----------------- | ------------- | --------------------------------------------------------------------------------- |
| **Plan Work**    | `plan-work.md`    | `gca-plan`    | Helps plan the immediate next steps or sub-tasks for the current working session. |
| **Work on Task** | `work-on-task.md` | `gca-work`    | Instructs GCA to implement a specific task from an implementation plan.           |
| **Fix Bug**      | `fix-bug.md`      | `gca-fix`     | A structured approach to diagnosing, reproducing, and fixing bugs with GCA.       |
| **Clarify**      | `clarify.md`      | `gca-clarify` | Asks GCA to explain complex code, confusing errors, or architectural decisions.   |

## 🔍 Code Review & Quality Gates

| Routine         | File             | Snippet      | Description                                                                                   |
| --------------- | ---------------- | ------------ | --------------------------------------------------------------------------------------------- |
| **Code Review** | `code-review.md` | `gca-review` | Initiates a comprehensive pre-merge review against `.gemini/styles/` and repo-specific rules. |

## 🚀 Deployment & Validation

| Routine                 | File                     | Snippet      | Description                                                                            |
| ----------------------- | ------------------------ | ------------ | -------------------------------------------------------------------------------------- |
| **Deploy to Dev**       | `deploy-to-dev.md`       | `gca-deploy` | Guides the deployment of changes to the development environment.                       |
| **Manual Verification** | `manual-verification.md` | `gca-verify` | Generates a manual testing checklist to verify feature behavior.                       |
| **Merge Prep**          | `merge.md`               | `gca-merge`  | Prepares the workspace for a clean merge, ensuring all logs and docs are synchronized. |
| **Check Log**           | `check-log.md`           | `gca-log`    | Analyzes application or CI/CD logs to identify deployment or runtime issues.           |

## 🔄 Agile & Session Management

| Routine               | File                   | Snippet      | Description                                                                                 |
| --------------------- | ---------------------- | ------------ | ------------------------------------------------------------------------------------------- |
| **Rehydrate Session** | `rehydrate-session.md` | `gca-resume` | Restores context from a previous session or handoff document.                               |
| **Sprint Wrapup**     | `sprint-wrapup.md`     | `gca-wrapup` | Summarizes completed work and updates agile trackers.                                       |
| **Record Lesson**     | `record-lesson.md`     | `gca-lesson` | Documents key learnings, edge cases, or architectural shifts discovered during development. |
| **Hardening Sprint**  | `hardening-sprint.md`  | `gca-harden` | Focuses GCA on security, performance, and reliability improvements.                         |

## 🤝 Handoffs

| Routine              | File                  | Snippet       | Description                                                                           |
| -------------------- | --------------------- | ------------- | ------------------------------------------------------------------------------------- |
| **Create Handoff**   | `handoff.md`          | `gca-handoff` | Generates a comprehensive handoff manifest for another developer or the next session. |
| **Execute Manifest** | `execute-manifest.md` | `gca-exec`    | Executes tasks defined in a handoff manifest.                                         |
| **Verify Handoff**   | `verify-handoff.md`   | `gca-check`   | Verifies that a handoff manifest is complete and actionable.                          |

---

## 💡 How to Use Routines

### Method 1: Using VSCode Snippets (Recommended)

1. Open the GCA Chat in VSCode.
2. Type the snippet prefix (e.g., `gca-work`).
3. Press `Enter` or `Tab` to expand the snippet.
4. Fill in any required variables (usually marked with brackets like `[TASK_DESCRIPTION]`) and submit.

### Method 2: Direct File Reference

1. Open GCA Chat.
2. Type `@workspace` to ensure workspace context.
3. Type the prompt, referencing the file: `Read @prompts/work-on-task.md and help me implement Task 3 from the impl plan.`
