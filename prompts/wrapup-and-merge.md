---
agent: agent
description: "Full end-of-sprint routine — wraps up the sprint, performs code review, and merges"
---

# Sprint Wrap-Up and Merge

Execute the complete end-of-sprint workflow by chaining the sprint wrap-up and merge routines.

## Phase 1 — Sprint Wrap-Up
Execute the instructions from `@prompts/sprint-wrapup.md` to document the completed work, update tasks, append to the change log, assess risks, and record lessons learned.

## Phase 2 — Merge (including Pre-Merge Review)
Once the sprint wrap-up is complete and the user is ready, execute the instructions from `@prompts/merge.md`.
*(Note: The merge routine inherently executes the code review routine, cleans the working tree, and handles committing and pushing.)*
