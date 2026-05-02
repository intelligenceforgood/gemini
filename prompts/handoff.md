---
agent: agent
description: "[Planner] Produce a Task Manifest ready to hand to the Executor"
---

# Handoff

Turn the current plan (from `/plan-work`, a sprint task, or the active todo list) into a self-contained **Task Manifest** the Executor can follow without replanning.

Read `.gemini/styles/manifest.md` for the full manifest schema and rules.

## Steps

1. **Confirm the source plan.** Use the active todo list, the last `gca-plan` output, or a `planning/tasks/*.md` sprint file. If none exists, tell the user to draft a plan first.

2. **Decide if a manifest is warranted.** Skip the manifest and tell the user to implement inline when ANY of these hold: estimated < 8 Agent turns, < 3 files, single repo with no migrations/env vars/API changes. The handoff+verify round-trip adds meaningful Planner cost and only pays back on longer work.

3. **Batch scope when possible.** If the active plan covers multiple sprints or phases that share files and verification, produce **one manifest covering the batch** rather than one-per-sprint. Manifest drafting cost is mostly fixed per session; two small manifests cost nearly twice as much as one combined one.

4. **Draft the manifest** using the template from `.gemini/styles/manifest.md`. Required sections:
   - `<contract>` — role, Agent mode status, version, scope, repos touched.
   - Goal (1–2 sentences).
   - Context (links only, do not inline standards).
   - `<files>` — files to modify, create, and explicitly NOT touch.
   - Step-by-step (numbered, each step independently testable).
   - `<do_not>` — explicit negatives to prevent drift.
   - `<verification>` — acceptance criteria as **runnable commands**, not prose.

5. **Quality check.** Before saving, verify:
   - Every file mentioned in steps appears in `<files>`.
   - Every acceptance criterion has a command the Agent can run.
   - No ambiguous phrasing ("handle errors gracefully", "improve performance"). Replace with concrete, testable requirements.
   - Architectural references link to `.gemini/styles/*.md` rather than restating.

6. **Save to disk.** Default path: `planning/handoffs/<YYYY-MM-DD>-<slug>.manifest.md`. Create the `handoffs/` folder if missing.

7. **Report.**
   - Print the manifest path.
   - Print a 3–5 bullet summary of scope.
   - Remind the user: **"Start a fresh GCA chat, ensure Agent Mode is ON, and ask me to execute the manifest."** A fresh session resets the context window, which meaningfully reduces token consumption for the execution phase.
