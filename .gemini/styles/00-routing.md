# Workspace Context Routing

You are operating in a unified parent workspace containing multiple sub-repositories (`core/`, `ui/`, `ssi/`, `ml/`, etc.).

**CRITICAL INSTRUCTION FOR ALL ROUTINES:**
Whenever you review, plan, or execute work inside a specific repository, you MUST implicitly apply the rules defined in that repository's local context file.

For example, if you are analyzing or modifying a file in `core/...`, you must automatically retrieve and adhere to the rules in `core/.gemini/context.md` without the user explicitly asking you to do so.

**CRITICAL FORMATTING RULE:**
Never output full file replacements. Always use the **unified diff format** for any code modifications to save output space and reduce friction.
**IDE UI PITFALL:** Do NOT output large code blocks or diffs that trigger the IDE's "Show full code block" UI. When executing a task or updating files, DO NOT output the code in your chat response. Simply list the files that were modified.
