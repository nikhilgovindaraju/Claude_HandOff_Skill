---
name: handoff-generator
description: Generates a structured handoff document so the user can continue this conversation in a different LLM (ChatGPT, Gemini, Cursor, Codex, or another Claude session) without losing context. ALWAYS use this skill when the user types "/handoff", says "handoff", "hand off", "session handoff", or asks to "continue this elsewhere", "switch to ChatGPT/Gemini/Cursor", "export this chat", "summarize for another LLM", "session is ending", "context limit", or anything similar. Also use when the user mentions running out of messages, hitting a session limit, or wanting to pick up the work later in a different tool. Do not skip this skill for short conversations — use the brief format instead.
---

# Handoff Generator

You are generating a handoff document. The user wants to continue this conversation in a different LLM (or a fresh Claude session) without re-explaining everything.

The handoff is read by the next LLM. It must be specific, accurate, and complete enough that the next LLM can pick up the work without asking the user to repeat themselves.

## Critical rules — follow all before writing anything

1. **Use ONLY information that explicitly appeared in this conversation.** Do not invent, infer, or fill gaps with plausible-sounding content. Hallucination is the #1 failure mode and ruins the handoff. If a section has nothing to populate, write "None established in this session."

2. **Be specific, not abstract.** "Switched from offset to cursor pagination because offset queries timed out past 10k rows" — yes. "Discussed pagination" — no.

3. **Preserve exact technical details.** Function names, file paths, error messages (verbatim if quoted), library versions, command strings, variable names. The next LLM needs these literally.

4. **Redact secrets.** Anything that looks like an API key, token, password, internal URL with auth, personal email, or private hostname → replace with `[REDACTED:type]`. When in doubt, redact.

5. **The "Tried and Ruled Out" section is the most important.** Without it, the next LLM will repeat past mistakes. Be thorough here even if it makes that section the longest.

6. **Do not summarize the chat chronologically.** Organize by the structure below. Order of conversation is irrelevant; current state is what matters.

## Decide which format to use

- **Long format** — use if the conversation has 5+ substantive exchanges, produced concrete decisions, code, or a clear problem being worked on.
- **Short format** — use if the conversation is brief, exploratory, or has not yet produced meaningful state.

If unsure, use long format.

## Long format — output exactly this structure

Output the handoff as a single markdown block, ready for the user to copy. Use these exact headers in this order:

```
# Session Handoff

## 1. Goal
One to three sentences. The overall thing the user is trying to accomplish. Distinguish overall goal from immediate sub-task if they differ.

## 2. Current State
What concretely exists right now:
- Files/code/artifacts created or modified (with paths if mentioned)
- What is working
- What is broken or incomplete, and how
- Exact error message or failure mode if there is one (verbatim)

## 3. Key Decisions Made
Bulleted list. Each item: decision + reason.
Only include decisions that were actually made, not options merely discussed.

## 4. Tried and Ruled Out (DO NOT REPEAT)
Most important section. Bulleted list. Each item: approach + why it failed or was rejected.
If nothing has been ruled out, write "Nothing ruled out yet."

## 5. Conventions and Constraints
What the next LLM must respect:
- Language/framework versions
- Style preferences explicitly stated
- Things the user said they will or won't do
- Naming conventions adopted mid-chat
- External constraints (deadlines, deployment limits, team rules)

## 6. Open Questions
Things not yet decided, or that need clarification. Phrased as actual questions.

## 7. Immediate Next Step
The single next action. Concrete. "Implement the retry logic in src/api/client.ts using exponential backoff" — not "continue the work."

## 8. Code Snapshot
Only if relevant. Latest version of code blocks central to current state, marked with filenames. Omit section entirely if no code is in play.

## 9. Continuation Prompt
A single paragraph the user pastes at the start of the next session, written in second person to the next LLM. Should reference the rest of the handoff and state the immediate ask. Example:
"You're picking up a session in progress. Read the handoff above. The codebase is a Next.js 14 app using Postgres and Drizzle. We're mid-implementation of cursor-based pagination on /api/posts and just hit a TypeScript error in the cursor decoder. Pick up at section 7."
```

## Short format — output exactly this structure

```
# Session Handoff (brief)

**Goal:** (1–2 sentences)

**Current state:** (what exists, what's working, what's broken)

**Tried and ruled out:** (anything already rejected — do not skip)

**Decisions:** (bulleted, with reasons)

**Next step:** (one concrete action)

**Continuation prompt:** (one paragraph to paste into the next LLM)
```

## After generating, do a self-check

Before returning the handoff to the user, silently verify:
- Did I invent anything not in the chat? Remove it.
- Is "Tried and Ruled Out" thorough?
- Are file paths, error messages, version numbers exact?
- Did I redact secrets?
- Is the Immediate Next Step a concrete action, not a vague direction?

If any check fails, fix it before responding.

## Final output

Return the handoff inside a single fenced markdown code block so the user can copy it with one click. After the code block, add one line: "Paste this into your next LLM session. Section 9 (or the Continuation prompt) is the actual prompt to lead with."

Do not add any preamble before the code block. Do not explain what you're doing. Just produce the handoff.
