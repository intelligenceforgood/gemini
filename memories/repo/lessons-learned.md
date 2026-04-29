# Lessons Learned

- **Destructive Updates to Trackers:** When instructed to "update" a markdown task list, Executors may overzealously rewrite the entire document. Always explicitly define the boundaries using `<do_not>` constraints. (Recorded during PhishDestroy Sprint 0.5 wrap-up).
- **Manual Action Deviations:** When the Executor makes intentional manual changes (like deleting an irrelevant file not listed in the manifest), it should communicate this in a "Notes from Executor" section to avoid being flagged as drift during the handoff-verify cycle.