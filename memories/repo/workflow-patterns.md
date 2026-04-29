# Workflow Patterns

## Planner/Executor lessons

- **Destructive Updates to Trackers:** When instructed to "update" a markdown task list or planning document to mark items as done, Executors may overzealously rewrite the entire document, truncating historical context and un-started tasks. They may also unilaterally delete other task files in the same directory.
  - **Mitigation:** Explicitly define the boundaries of the update. Use `<do_not>` rules such as: "Do not truncate, summarize, or remove un-started tasks when updating checklists" and "Do not delete any files not explicitly listed in `<files>`."