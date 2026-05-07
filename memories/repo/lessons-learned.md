# Lessons Learned

- **Destructive Updates to Trackers:** When instructed to "update" a markdown task list, Agents may overzealously rewrite the entire document. Always explicitly define the boundaries using `<do_not>` constraints. (Recorded during PhishDestroy Sprint 0.5 wrap-up).
- **Manual Action Deviations:** When the Agent makes intentional manual changes (like deleting an irrelevant file not listed in the task plan), it should communicate this in a "Notes from Agent" section to avoid being flagged as drift during the sprint wrap-up or merge review.
- **Mocking Background Tasks in Integration Tests:** When testing API endpoints that trigger long-running background tasks, ensure the task handler itself is mocked or patched out to avoid tests silently taking several minutes instead of milliseconds.
- **Executing Multi-Phase Routines:** When instructed to execute a multi-phase prompt/routine (like merge.md), do not pause and ask for permission to proceed to the next phase unless explicitly instructed to do so. Complete all phases of the routine autonomously to avoid halting the workflow.
- **Task Plan Maintenance:** When working from a task plan or manifest, always explicitly check off completed tasks in the original markdown document (e.g. changing `[ ]` to `[x]`) to keep the project state accurate.
- **Interpreting Prompt Files as Actions:** When a user provides a prompt file (like `merge.md`) alongside a target file, they intend for you to execute the prompt's instructions against the context, NOT literally append the prompt's content into the target file.
