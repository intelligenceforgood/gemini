---
agent: agent
description: "[Planner] Break a feature or task into actionable steps"
---

# Plan Work

**Role: Planner.** Take a feature request, task description, or user story and break it into implementable steps. This routine produces a plan; it does not implement. **Always save the resulting plan document as a new markdown file in the `planning/tasks/` directory.** When the plan is ready, the user will either continue here (for small tasks) or run `/handoff` to produce a manifest for an Executor to pick up.

## Steps

1. **Clarify scope.** Read the task description (from planning/, a PR, or user input). If it's ambiguous, list assumptions and confirm with the user.

2. **Identify affected repos.** Determine which repos need changes (core/, ui/, ssi/, infra/, etc.) and what kind of changes (API, UI, database, infrastructure).

3. **Check architecture.** Ask the user to explicitly tag `@file:.gemini/styles/architecture.md` and the relevant language styleguide (e.g., `@file:.gemini/styles/python.md`) before generating the plan. Look for relevant patterns, especially:
   - Request routing (UI → API proxy → FastAPI)
   - Store/factory patterns
   - Worker/job patterns for background tasks

4. **Break into steps.** Draft the plan using the template structure below. Create a numbered task list:
   - Order by dependency (database first, then API, then UI)
   - Each step should be independently testable
   - Flag steps that require manual actions (migrations, deploys)
   - **CRITICAL**: Each step MUST explicitly list the exact files to be modified using `@file:path/to/file` or `@folder:path/to/folder` tags to ensure subsequent execution is tightly scoped.

5. **Identify risks.** Note:
   - Breaking changes to existing APIs or schemas
   - New environment variables (need docs + tests)
   - Database migrations (need careful sequencing)

6. **Track with todos.** Create a todo list to track progress through the steps.

7. **Decide the handoff.** Apply the skip-threshold heuristic:
   - **Skip `/handoff`, implement inline** if ANY of these hold: estimated < 8 Executor turns, < 3 files touched, single repo with no migrations/env vars/API changes, or throwaway/exploratory work. The Planner-overhead tax (roughly 2× the Planner cost for handoff + verify) only pays back on longer work.
   - **Use `/handoff`** for multi-file, multi-repo, or risky work where scope discipline matters more than speed.
   - **Batch sprints into one manifest** when possible. Don't produce one manifest per sprint when two or three sprints share a coherent slice of work — the Planner cost is mostly fixed per handoff session, so one larger manifest is far cheaper than N small ones.

## Template

**Objective:** {Brief description of the goal}

### 1. Milestones

Divide the work into logical phases (e.g., Database, Backend, Frontend).

### 2. Task Checklist

Create a markdown task list `- [ ]` for each step. Each task should be independently testable.

**CRITICAL:** Every task MUST explicitly mention the exact files it will modify using `@file:path/to/file` or `@folder:path/to/folder` tags to restrict future execution scope.

- [ ] Step 1: ...
- [ ] Step 2: ...