---
agent: agent
description: "Rehydrate session context by loading architecture, recent changes, and lessons learned"
---

# Rehydrate Session

LLMs are stateless between sessions. This routine forces the model to read the critical workspace state and repo memory so it operates with full "senior engineer" context before starting new work.

## Steps

1. **Read Memories.** Silently read `/memories/repo/lessons-learned.md`. Pay special attention to past pitfalls and workflow corrections (e.g., the necessity of explicit `@file` tagging).
2. **Read Architecture.** Silently read `.gemini/styles/architecture.md` to understand the routing, auth, and database patterns.
3. **Read Recent Changes.** Silently read `planning/change_log.md` (the latest entries) to understand what was recently completed.
4. **Load Workflow Rules.** Silently check for any overarching workflow rules in `.gemini/styles/workflow.md` (if it exists).
5. **Enforce Guardrails.** Acknowledge the context loaded. Explicitly remind the user to use strict `@file` and `@folder` tags for any subsequent execution requests to prevent hallucinations.
6. **Ready.** Report that the session is fully rehydrated and ask the user what task they want to work on next.
