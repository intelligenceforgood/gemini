---
agent: agent
description: "Save a lesson, pattern, or pitfall to repo memory for future sessions"
---

# Record Lesson

Capture something learned during this session so future sessions benefit from it.

## Steps

1. **Identify the lesson.** Ask the user what they want to record, or infer from the current conversation context.

2. **Categorize:**
   - **Coding pitfall** — a mistake or anti-pattern to avoid
   - **Architecture pattern** — an approach that works well in this codebase
   - **Workflow tip** — a useful command, shortcut, or process improvement
   - **Environment/config** — a configuration insight or env var behavior

3. **Write to repo memory.** Add to `/memories/repo/lessons-learned.md` under the appropriate section. Format as a concise bullet point:
   - What the lesson is (one line)
   - Why it matters or what goes wrong without it (one line)
   - Example if helpful (code snippet or command)

4. **Auto-Promote.** If the lesson is a critical workflow guardrail (e.g., GCA hallucinated due to a lack of strict `@file` tags) OR the same category now has 3+ similar lessons, **do not wait for permission**. Automatically edit and promote the pattern to the appropriate style guide in `.gemini/styles/` (such as `workflow.md` or `python.md`).

5. **Confirm.** Tell the user what was recorded to the lessons-learned file, and explicitly state if it was also automatically promoted to a permanent style guide.
