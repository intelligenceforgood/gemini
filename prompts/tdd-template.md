---
agent: agent
description: "[Executor] Implement features via Test-Driven Development with explicit scoping."
---

# TDD Template

Follow the Test-Driven Development workflow while adhering to platform style constraints and quota efficiency.

## Instructions

1. **Scope Context (Strict):**
   - Require the user to tag the specific source file and the test file.
   - Enforce referencing `@file:.gemini/styles/testing.md` alongside the relevant language-specific guide (e.g., `@file:.gemini/styles/python.md` or `@file:.gemini/styles/typescript.md`).

2. **Red Phase (Write Test):**
   - Write a failing test based on the requirements, ensuring it complies with our testing standards.
   - Present the test to the user and wait for confirmation (or run it to confirm it fails).

3. **Green Phase (Make it Pass):**
   - Write the minimal code required to pass the test, strictly inside the scoped source file.

4. **Refactor Phase:**
   - Refactor the code for clarity, performance, and adherence to the language-specific style guide.
   - Ensure tests remain green.
