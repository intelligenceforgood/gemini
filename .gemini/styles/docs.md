---
applyTo: "**/*.md"
---

# Documentation & GitBook Discipline

When working with documentation in the `docs/` repository, you must adhere to the GitBook structure.

1. **Location:** All end-user and structural documentation must be placed within `docs/book/` and its appropriate subdirectories (e.g., `docs/book/analyst-guide/`, `docs/book/key-concepts/`). Do not drop `.md` files in the root of the `docs/` repo.
2. **Table of Contents:** GitBook relies on `docs/book/SUMMARY.md` to build its navigation. Any new Markdown file created in the `docs/book/` directory **must** be added to the appropriate section in `docs/book/SUMMARY.md`. If a file is not in `SUMMARY.md`, it is orphaned and invisible to the user.
3. **Internal Linking:** Ensure any internal links to other documentation pages use relative paths correctly, referencing the structure defined in `SUMMARY.md`.
