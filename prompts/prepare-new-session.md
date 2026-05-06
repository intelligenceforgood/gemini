---
agent: agent
description: "Summarize the current session and generate a prompt to kick off the next fresh session"
---

# Prepare New Session

We have done a lot of work in this session and the context window is getting large. We need to start a fresh chat to maintain focus, avoid attention dilution, and save tokens.

## Steps

1. **Summarize current state:** Briefly list what was successfully accomplished in this session.
2. **Identify next steps:** Identify the immediate next task that needs to be tackled from the active plan or current state of work.
3. **Determine target files (Strict Scope):** Identify the 1-5 exact files (code, tests, relevant `.gemini/styles/*.md`, or active plan) that the new session will need to use to continue the work. Do not include files that are already completed and no longer need edits.
4. **Generate Session Prompt:** Create a ready-to-paste prompt for the user. It should instruct the new session on what context to load, what the goal is, and explicitly tag the required files using `@file:`.

## Output Format

Present a brief summary of the session, then provide the copy-paste prompt formatted like this:

**Copy and paste this into a new chat session:**

```text
We are continuing work from a previous session. We just completed: [Brief Summary].

Your immediate next step is to: [Next Step].

Please strictly scope your context and execute this next step using the following files:
@file:path/to/file1
@file:path/to/file2
```
