---
agent: agent
description: "[Executor] Fix a reported bug with explicitly scoped context."
---

# Fix Bug

Diagnose and fix the reported issue within a narrow, optimized context.

## Instructions

1. **Scope Context (Strict):**
   - Require the user to tag the specific files involved with the bug.
   - Ask the user to tag the appropriate language style guide (e.g., `@file:.gemini/styles/python.md` or `@file:.gemini/styles/typescript.md`).
   - **Crucially:** If the bug relates to testing, pipelines, deployment, or vulnerabilities, enforce the inclusion of `@file:.gemini/styles/testing.md`, `@file:.gemini/styles/ci-cd.md`, and/or `@file:.gemini/styles/security.md`.

2. **Diagnose:**
   - Review the provided logs, stack traces, or descriptions.
   - Identify the root cause without triggering wide workspace scans.

3. **Execute Fix:**
   - Implement the bug fix.
   - Comply fully with the architectural and security standards loaded in the context.

4. **Verify:**
   - Ensure the fix addresses the issue. Provide a concise explanation of what was changed and why.
   - Provide a run command to test the fix if applicable.
