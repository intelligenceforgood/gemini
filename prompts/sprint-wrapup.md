---
agent: agent
description: "End of sprint — update plans, log changes, assess merge readiness"
---

# Sprint Wrap-Up

Close out a sprint or work session with proper documentation and a clean save point.

## Steps

1. **Verify fidelity and drift.** Before closing the sprint, inspect the `git diff` for drift. Look specifically for:
   - Unrelated refactors or stylistic edits.
   - New dependencies, new env vars, or new files that were not authorized.
   - Silent API or schema changes.
   - If drift is found, alert the user so they can revert or accept it.

2. **Identify completed work.** Check `git log` across repos since the sprint started. Summarize what was implemented, fixed, or changed.

3. **Update task checkboxes.** Find the active implementation plan in `planning/tasks/` and mark completed tasks as `- [x]`.

4. **Update change log.** Append a dated summary to `planning/change_log.md`:
   - What changed (features, fixes, refactors)
   - Which repos were affected
   - Any env var or config changes

5. **Manual steps for the user.** List any steps that require manual execution:
   - Alembic migrations: `i4g db migrate dev` / `i4g db migrate prod`
   - Docker image builds and which images
   - Cloud Run deploys
   - Include exact commands.

6. **Risk assessment.** Identify risks of breaking existing functionality:
   - API contract changes
   - Database schema changes
   - New env vars that need setting in production
   - Quick validation tests the user can run

7. **Merge readiness.** State whether this is a good merge point. List blockers or caveats.

8. **Record lessons.** If anything was learned (patterns, pitfalls, workflow improvements), add to `/memories/repo/lessons-learned.md`.
