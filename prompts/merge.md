---
agent: agent
description: "Full review + merge — runs code review then commits and pushes all changed repos to main"
---

# Merge

Run the full pre-merge review, then commit and push all changed repos across the entire workspace to `main`.
This is the single combined routine — no need to run `/code-review` separately.

## Phase 1 — Identify Changed Repositories

1. **Scan the Workspace:** Check all sibling repositories in the parent directory (`../`) using `git status` to identify which repositories have uncommitted or staged changes. 
   - Because the platform is implemented by multiple repos, they must be merged together to ensure the application remains in a consistent state.
2. List all identified changed repositories before proceeding.

## Phase 2 — Pre-Merge Review

For all changed repos identified in Phase 1:
Execute every step of the code review routine (`@prompts/code-review.md`):
code audit, quality gates, tests, docs/config check. Fix all issues found before proceeding.

Do NOT proceed to Phase 3 if any quality gate fails or tests have failures in ANY repo.

## Phase 3 — Clean Working Tree

After the review passes, verify the working tree is merge-ready for EVERY changed repo:

1. **No stray files.** `git status` in each changed repo shows only intentional changes.
   - No merge conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
   - No temp files (`*.tmp`, `*.bak`, `*.swp`, `.DS_Store`, `__pycache__/`)
   - No debug scripts outside `tests/` (`debug_*.py`, `scratch_*.py`)
   - If any found, alert the user and do NOT proceed

2. **No secrets.** Scan staged content for:
   - API keys / tokens (`sk-`, `ghp_`, `AIza`, `AKIA`)
   - Private keys (`BEGIN RSA PRIVATE KEY`, etc.)
   - `.env` files that should be gitignored
   - Hard-coded local paths (`/Users/<name>/`)
   - If any found, alert the user and do NOT proceed

## Phase 4 — Commit and Push

Follow `.gemini/styleguide.md` for all commit hygiene rules.

For each changed repo identified in Phase 1, execute the following to ensure they are merged simultaneously:

```bash
cd <repo-root>
git add -A
git commit -m "<conventional commit message>"
git push origin main
```

- Use conventional commit format: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`
- Generate descriptive commit messages from the changes — do NOT ask the user to confirm
- If push fails (e.g., diverged), alert the user — do NOT force push

## Phase 5 — Summary

Report per-repo: issues found + fixed, test results, files committed, commit hash, push status.
