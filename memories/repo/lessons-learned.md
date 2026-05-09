# Lessons Learned

## Workspace Navigation & Token Conservation

- **Token Usage Optimization:** Do not needlessly read the entire workspace root if targeted context is sufficient. However, understand the overall architecture by using search tools strategically (`grep`, file listing) rather than reading massive files to conserve daily token quotas.
- **Avoid Hallucinated Folders:** Always verify the correct absolute or relative path before creating or modifying files. For instance, do not create a file under `docs` when it belongs in a project-specific `docs` folder.

## Boundaries and Reliability

- **Agent Mode Necessity:** Relying on "Agent Mode" is crucial for consistent behavior. Without it, behavior can become erratic, refusing tasks or hallucinating user instructions (e.g., claiming the user restricted access). If instructed to execute a task, perform it directly or explain the specific technical block.
- **Uncertainty & Asking for Clarification:** If a user's prompt does not make sense, or you are unsure about the safety or correctness of an action, **STOP and ask the user for clarification**. Do not proceed with harmful actions just to satisfy the prompt.
- **Architect-Level Standards:** Consistently follow the established architecture, quality guidelines, and standards across all I4G repositories.

## Workflow and Prompt Interpretation

- **Avoiding "Show full code block" UI:** Do not output large code blocks or full file diffs in the chat, as it triggers an annoying IDE wrapper. If files are already updated via tools or routines, simply list the names of the modified files.
- **Manual Action Deviations:** When the Agent makes intentional manual changes (like deleting an irrelevant file not listed in the task plan), it should communicate this in a "Notes from Agent" section to avoid being flagged as drift during the sprint wrap-up or merge review.

## Tracking & Planning

- **Destructive Updates to Trackers:** When instructed to "update" a markdown task list or planning document, do not rewrite the entire document or truncate historical context and un-started tasks. Always explicitly define boundaries using `<do_not>` constraints.
- **Task Plan Maintenance:** When working from a task plan or manifest, always explicitly check off completed tasks in the original markdown document (e.g., changing `[ ]` to `[x]`) to keep the project state accurate.

## Coding Specifics

- **Interface/Protocol Mismatches in Plans:** Users sometimes prescribe implementation details that violate existing protocol contracts. Safely deviate from the plan to honor the existing structural contract rather than breaking adjacent interfaces.
- **Hallucinated File Modifications:** Verify `git diff` to ensure required files were actually modified before merging. Agents may hallucinate completing UI tasks by checking them off without modifying source files.
- **Alembic Auto-generator Template Errors:** The auto-generator may produce template-rendering errors. Explicitly write valid boilerplate migration scripts to bypass the error if it occurs.
- **Test Fixture Constraints:** When mocking test models (e.g., SQLite NOT NULL constraints), ensure test fixtures populate required fields.
- **Mocking Background Tasks in Integration Tests:** When testing API endpoints that trigger long-running background tasks, ensure the task handler itself is mocked or patched out to avoid tests silently taking several minutes.
- **Next.js 15 Dynamic Routing:** Dynamic route `params` (and `searchParams`) are asynchronous in Next.js 15. They must be typed as `Promise<{ [key]: string }>` and awaited before use.
