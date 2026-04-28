# The Handoff Prompt

## Primary version (use for any chat longer than ~5 exchanges)

```
You are about to generate a HANDOFF document so this conversation can continue in a different LLM (Claude, ChatGPT, Gemini, Cursor, or another tool) without losing context.

CRITICAL RULES — read all before writing anything:

1. Use ONLY information that explicitly appeared in this conversation. If a section has no information, write "None established in this session." Do NOT invent, infer, or fill gaps with plausible-sounding content. Hallucination here is the #1 failure mode and ruins the handoff.

2. Be specific, not abstract. "Switched from offset to cursor pagination because offset queries timed out past 10k rows" — yes. "Discussed pagination" — no.

3. Preserve exact technical details: function names, file paths, error messages (verbatim if quoted), library versions, command strings, variable names. The next LLM needs these literally.

4. Redact anything that looks like a secret: API keys, tokens, passwords, internal URLs with auth, personal emails, private hostnames. Replace with [REDACTED:type]. When in doubt, redact.

5. The "Tried and Ruled Out" section is the most important section in this document. The next LLM will repeat past mistakes without it. Be thorough here even if it makes that section the longest.

6. Do not summarize the chat chronologically. Organize by the structure below. Order of conversation is irrelevant; current state is what matters.

7. If this conversation is very short or has not produced meaningful decisions, say so honestly in a one-paragraph "Minimal handoff" note instead of padding empty sections.

OUTPUT FORMAT — use these exact headers, in this order, as a markdown document:

# Session Handoff

## 1. Goal
One to three sentences. What is the user ultimately trying to accomplish? Distinguish the overall goal from the immediate sub-task if they differ.

## 2. Current State
What concretely exists right now. Be precise:
- What code/files/artifacts have been created or modified (with paths if mentioned)
- What is working
- What is broken or incomplete, and how
- The exact error message or failure mode if there is one (verbatim)

## 3. Key Decisions Made
Bulleted list. Each item: the decision + the reason. Examples:
- Chose Postgres over MongoDB because the data is heavily relational.
- Using Zod for validation, not Yup — Yup was dropped due to TypeScript inference issues.
Only include decisions that were actually made, not options that were merely discussed.

## 4. Tried and Ruled Out (DO NOT REPEAT)
The most important section. Bulleted list. Each item: the approach + why it failed or was rejected. Examples:
- Tried using Promise.all for parallel fetches — hit rate limits, switched to a queue.
- Considered storing sessions in Redis — rejected because the deployment target doesn't allow stateful services.
If nothing has been ruled out, write "Nothing ruled out yet."

## 5. Conventions and Constraints
Anything the next LLM must respect:
- Language/framework versions
- Style preferences explicitly stated by the user
- Things the user said they will or won't do
- Naming conventions adopted mid-chat
- External constraints (deadlines, deployment limits, team rules)

## 6. Open Questions
Things the user has not yet decided, or things that need clarification. Phrased as actual questions.

## 7. Immediate Next Step
The single next action. Be concrete. "Implement the retry logic in src/api/client.ts using the exponential backoff approach we sketched" — not "continue the work."

## 8. Code Snapshot (only if relevant)
If specific code blocks are central to the current state, include the latest version of each. Mark each with its filename. Do not include code that is no longer relevant. If no code is in play, omit this section entirely.

## 9. Continuation Prompt
A single paragraph the user can paste at the start of the next session, written in second person to the next LLM. It should reference the rest of this handoff and state the immediate ask. Example:
"You're picking up a session in progress. Read the handoff above. The codebase is a Next.js 14 app using Postgres and Drizzle. We're mid-implementation of cursor-based pagination on the /api/posts endpoint and just hit a TypeScript error in the cursor decoder. Pick up at section 7 — Immediate Next Step."

END OF FORMAT.

Self-check before you finish:
- Did you invent anything not in the chat? Remove it.
- Is "Tried and Ruled Out" thorough?
- Are file paths, error messages, and version numbers exact?
- Did you redact secrets?
- Is the Immediate Next Step a concrete action, not a vague direction?

Now generate the handoff.
```

## Short version (for chats under ~5 exchanges)

```
Generate a brief handoff so I can continue this in another LLM. Use only what appeared in this conversation — do not invent details. Format:

**Goal:** (1–2 sentences)
**Current state:** (what exists right now, what's working, what's broken)
**Tried and ruled out:** (anything we already rejected — critical, do not skip)
**Decisions:** (bulleted, with reasons)
**Next step:** (one concrete action)
**Continuation prompt:** (one paragraph to paste into the next LLM)

Redact any secrets. If a section is empty, write "None."
```

## Usage notes

- Paste the **primary version** when you've had a substantive working session.
- Paste the **short version** for quick brainstorms or early-stage chats.
- After the LLM generates the handoff, skim section 4 ("Tried and Ruled Out") specifically — that's where hallucination is most damaging and most subtle.
- For destination LLMs that are stricter about format (Cursor, Codex), the markdown structure transfers cleanly.
- If the output looks rushed or sections are thin at the end, just reply: "Section X feels thin. Expand it using only what's in this chat." Models often recover well on a second pass.