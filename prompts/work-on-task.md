---
agent: agent
description: "[Executor] Execute a specific task with explicitly scoped context."
---

# Work on Task

Execute the given task efficiently while adhering to quota-saving strict file scoping.

## Instructions

1. **Scope Context (Strict):**
   - **CRITICAL:** Do NOT scan or search the workspace.
   - If the user has not explicitly tagged the target files (using `@file` or `@folder`) or does not have them open in their active editor tabs, **STOP** immediately and ask them to do so. Do not guess.
   - Instruct the user to explicitly tag the relevant tech stack's style guide if they haven't already:
     - For Python: Tag `@file:.gemini/styles/python.md`
     - For TypeScript/React: Tag `@file:.gemini/styles/typescript.md`

2. **Execute:**
   - Make the necessary code modifications strictly within the tagged files.
   - Ensure the new code follows the rules defined in the explicitly tagged style guides.

3. **Verify:**
   - Keep explanations lean and ensure no conversational fluff is introduced.
