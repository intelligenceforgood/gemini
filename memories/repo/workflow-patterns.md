# Workflow Patterns

## Planner/Executor lessons

- **Destructive Updates to Trackers:** When instructed to "update" a markdown task list or planning document to mark items as done, Executors may overzealously rewrite the entire document, truncating historical context and un-started tasks. They may also unilaterally delete other task files in the same directory.
  - **Mitigation:** Explicitly define the boundaries of the update. Use `<do_not>` rules such as: "Do not truncate, summarize, or remove un-started tasks when updating checklists" and "Do not delete any files not explicitly listed in `<files>`."
- **Interface/Protocol Mismatches in Manifest:** Planners sometimes prescribe implementation details (e.g., "yield records") that violate existing protocol contracts (e.g., expecting a dict return).
  - **Mitigation:** The Executor should safely deviate from the manifest to honor the existing structural contract rather than breaking adjacent interfaces. The Planner must inspect target interfaces before prescribing return types.
- **Hallucinated File Modifications:** Executors may hallucinate completing UI or template tasks by checking them off in the tracker and changelog without actually modifying the source code files (e.g. creating wireframe markdown instead of updating `.tsx` files).
  - **Mitigation:** Always verify the `git diff` to ensure the required files in `<files>` were actually modified before accepting a handoff. Explicitly instruct Executors to ensure all requested code files are modified and committed.
- **Alembic Auto-generator Template Errors:** The auto-generator may produce template-rendering errors (e.g., `<mako.runtime.Undefined object>`), causing invalid python scripts.
  - **Mitigation:** Instruct the Executor to explicitly write valid boilerplate migration scripts to bypass the error if it occurs.
- **Test Fixture Constraints:** When mocking `threat_actors` (e.g., in a `_seed_actor` helper) in tests, SQLite enforces NOT NULL constraints on fields like `display_name`.
  - **Mitigation:** Ensure test fixtures populate a mock `display_name` to satisfy the constraint.