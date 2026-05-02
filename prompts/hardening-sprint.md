---
agent: agent
description: "Bootstrap a platform hardening session — load plan, check progress, route to correct model"
---

# Hardening Sprint

Start a platform hardening work session. Load the review, execution plan, and guide you to the next task.

## Steps

1. **Load the plan and execution state.** Read these files:
   - `planning/tasks/platform-review-2026-04-17.md` — the full architecture review (Parts 1–6)
   - `planning/tasks/platform-hardening-execution.md` — the execution plan with checkboxes

2. **Identify progress.** Count checked (`- [x]`) vs unchecked (`- [ ]`) tasks. Report:
   - Current phase (the earliest phase with unchecked tasks)
   - Next unchecked task
   - Any blocked tasks (dependencies not yet complete)
   - Updated progress table

3. **Check Agent Mode requirements.** Read Part 6 of the review ("Execution Guidance"). For the next task:

   ⚠️ **If the task is marked as complex or multi-file**, warn the user:

   > **Quota Management Notice:** Task {id} involves {reason}. Ensure Agent Mode is ON if autonomous execution is required, but be mindful of your daily quota.

   If the task is straightforward or single-file, remind the user that Agent Mode can stay OFF.

4. **Load relevant context.** Based on the next task's repo(s), read:
   - The relevant source files referenced in the review finding
   - The relevant files in `.gemini/styles/` if the task touches Core/SSI/UI integration
   - Repo memory (`memories/repo/`) for lessons learned

5. **Present the work plan.** Show:
   - Task ID, title, repo, effort, model recommendation
   - The specific finding from the review (with file references)
   - Acceptance criteria from the execution plan
   - Dependencies (met or unmet)
   - Suggested implementation approach

6. **Confirm and execute.** Ask the user to confirm, then use the `work-on-task` routine pattern:
   implement → test → document → validate → summarize.

7. **After completion.** Check off the task in the execution plan, update the progress table,
   and note the change in `planning/change_log.md`.
