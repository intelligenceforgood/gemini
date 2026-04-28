# Customizing the GCA Framework

The GCA Productivity Framework is designed to be extensible. As the I4G platform evolves, so should our standards and routines.

## Adding a New Routine

If your team develops a new workflow that should be standardized, add it to the framework:

1.  **Create the Prompt**:
    *   Create a new markdown file in the `gemini/prompts/` directory (e.g., `gemini/prompts/load-testing.md`).
    *   Use clear, imperative language. Structure the prompt to give GCA context, an objective, and a specific output format.
    *   Example structure:
        ```markdown
        # Context
        We need to perform load testing on a specific service.

        # Instructions
        1. Analyze the provided endpoints.
        2. Generate a Locust (`locustfile.py`) script.
        3. Include varied user behavior simulation.

        # Output Format
        Provide the complete python code in a single code block.
        ```
2.  **Update the Snippets**:
    *   Open `gemini/snippets/gemini.code-snippets`.
    *   Add a new JSON entry for your prompt to make it easily accessible in VSCode.
3.  **Update the Catalog**:
    *   Add your new routine to `docs/routine-catalog.md` so other developers know it exists.

## Updating the Styleguide

The `.gemini/styleguide.md` file is the master document for architectural and coding standards.

1.  **Edit `styleguide.md`**: When introducing a new technology (e.g., switching from `pytest` to `ward`, or updating a UI library), update the relevant section in `.gemini/styleguide.md`.
2.  **Commit and Push**: Ensure these changes are reviewed via PR, as they will affect the behavior of GCA for all developers globally once pulled.
3.  **Propagation**: Because project repositories use symlinks to reference this file, updates will immediately apply to all local developer environments the next time they pull the `gemini` repository.

## Repo-Specific Overrides (Local Context)

While `gemini/` holds global standards, individual repositories often have unique requirements.

Instead of polluting the global `styleguide.md`, add a `.gemini/context.md` or update the local `.gemini/config.yaml` *inside the specific repository* (e.g., `ui/.gemini/context.md`).

When GCA is active in that repository, it will read both the symlinked global styleguide and the local context files, merging the instructions.

**Example `core/.gemini/context.md`:**
```markdown
# Core Repo Specifics
- Always use the `i4g-backend` conda environment.
- Run migrations using `alembic upgrade head` before running tests.
- This repository interacts heavily with BigQuery; prioritize optimized SQL over ORM methods for analytics queries.
```
