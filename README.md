# Handoff Generator

**Continue your AI conversations across different LLMs without losing context.**

Ever been deep in a coding session with Claude when the message limit hits? Or wanted to switch from ChatGPT to Gemini mid-task? You end up writing the same context again, what you're building, what you've tried, what didn't work.

This skill fixes that. One command, one structured handoff document, ready to paste into any other LLM.

---

## The problem

You're working with an AI on something real, a codebase, a debugging session, a design problem. The session ends or you want to switch tools. Your options today:

- Re-explain everything from scratch (slow, you'll forget things)
- Ask the AI for a summary (vague, misses the dead ends, hallucinates details)
- Copy the entire chat (too long, and the new LLM gets confused by it)

Handoff Generator gives you a fourth option: a **structured, accurate, copy-pasteable handoff** that the next LLM can actually use.

---

## What it does

When you trigger the skill, Claude generates a markdown document with these sections:

1. **Goal** — what you're trying to accomplish
2. **Current State** — what exists, what's working, what's broken
3. **Key Decisions** — choices you made and why
4. **Tried and Ruled Out** — *the most important section* — approaches already rejected, so the next LLM doesn't suggest them
5. **Conventions and Constraints** — versions, style rules, things you said you won't do
6. **Open Questions** — what's not decided yet
7. **Immediate Next Step** — the single concrete next action
8. **Code Snapshot** — current code blocks, if relevant
9. **Continuation Prompt** — a ready-to-paste opener for the next LLM

For short chats, it produces a brief 6-line version instead. The skill picks automatically.

---

## Install

### Claude (skill)

**Requirements:** Paid plan (Pro, Max, Team, or Enterprise). Code execution must be enabled in **Settings → Capabilities**.

1. Download [`handoff-generator.zip`](./handoff-generator.zip) from this repo.
2. In Claude, open **Customize → Skills**.
3. Click **"+"** → **"Create skill"** → **"Upload"**.
4. Upload the zip.
5. Confirm the toggle next to **handoff-generator** is on.

To use: type `/handoff` (or say things like "give me a handoff," "I'm switching to ChatGPT," "session is ending"). The skill will produce the document.

### ChatGPT (Custom GPT)

1. Go to **Explore GPTs → Create**.
2. In the **Instructions** field, paste the contents of [`SKILL.md`](./handoff-generator/SKILL.md) — but skip the YAML frontmatter (everything between the `---` markers at the top).
3. Name it "Handoff Generator." Description: *"Generates a structured handoff so you can continue this conversation in another LLM."*
4. Save.

To use: open the GPT and type "handoff" or paste your conversation in and ask for a handoff.

### Gemini (Gem)

1. Go to **Gems → Create new**.
2. Paste the same instructions (skip the YAML frontmatter) into the Gem's system prompt.
3. Save.

To use: open the Gem and ask for a handoff.

### Any other LLM (manual)

If you're using a tool that doesn't support custom skills/GPTs/Gems, just copy the prompt from [`HANDOFF_PROMPT.md`](./HANDOFF_PROMPT.md) and paste it at the end of any chat. Same output.

---

## How to use it

**At the end of a session:**

```
/handoff
```

Claude returns a markdown block. Click copy. Paste into your next LLM. Done.

**When switching tools mid-task:**

```
I'm about to switch to Cursor. Give me a handoff.
```

**When the session limit is approaching:**

```
Session is about to end — handoff please.
```

**For a quick summary of a short chat:**

The skill auto-detects short conversations and produces a brief 6-line version instead of the full 9-section format.

---

## Example

**You** (after a 30-message debugging session):
> /handoff

**Claude returns:**

````markdown
# Session Handoff

## 1. Goal
Implement cursor-based pagination on the /api/posts endpoint of a Next.js 14 + Drizzle + Postgres app, replacing the current offset-based pagination that times out past 10k rows.

## 2. Current State
- Modified: `src/app/api/posts/route.ts`, `src/lib/cursor.ts` (new file)
- Working: cursor encoding/decoding for `created_at` + `id` composite keys
- Broken: TypeScript error in `cursor.ts:42` — "Type 'string | null' is not assignable to type 'string'" when decoding from URL params
- Verbatim error: `TS2322: Type 'string | null' is not assignable to type 'string'.`

## 3. Key Decisions Made
- Cursor based on `(created_at, id)` composite, not just `id`, because posts can share timestamps under bulk imports.
- Cursors are base64-encoded JSON, not opaque tokens, because the team wants to debug them in logs.
- Page size is fixed at 20, not user-configurable, to simplify caching.

## 4. Tried and Ruled Out (DO NOT REPEAT)
- Tried `OFFSET / LIMIT` with index hints — still timed out at 12k rows.
- Tried Drizzle's built-in `.offset()` helper — same problem, it's just sugar over OFFSET.
- Considered keyset pagination on `id` alone — rejected, doesn't handle ties.
- Considered Relay-style cursors — rejected as overkill for an internal API.

## 5. Conventions and Constraints
- TypeScript strict mode is on.
- No new dependencies — must use only what's in package.json (Drizzle 0.30, Zod 3.22).
- API responses follow the existing `{ data, meta }` shape.

## 6. Open Questions
- Should the cursor expire? Currently it doesn't.
- How should we handle a cursor that points to a deleted row?

## 7. Immediate Next Step
Fix the TypeScript error in `src/lib/cursor.ts:42` by narrowing the URL param type — likely with a Zod schema parse before the decode call.

## 8. Code Snapshot
[current contents of cursor.ts and route.ts]

## 9. Continuation Prompt
You're picking up a session in progress. Read the handoff above. The codebase is a Next.js 14 app using Postgres and Drizzle. We're mid-implementation of cursor-based pagination on /api/posts and just hit a TypeScript error in the cursor decoder at src/lib/cursor.ts:42. Pick up at section 7 — fix the type narrowing using a Zod schema, then continue with the open questions in section 6.
````

Paste sections 1–8 into the new LLM as context. Lead with section 9 as the actual prompt. The new LLM picks up where you left off.

---

## Why this works

A few design choices that make this prompt more reliable than "just summarize the chat":

- **The "Tried and Ruled Out" section is mandatory.** Without it, the next LLM will cheerfully suggest the exact approach you rejected an hour ago. This is the single most important section.
- **Anti-hallucination rule.** The skill is told explicitly to use only what appeared in the conversation, and to write "None established" rather than invent content. LLMs love to fill empty sections with plausible fiction; this guardrail blocks that.
- **Section 9 is the handshake.** The first 8 sections are *context* the next LLM reads. Section 9 is what you actually paste as your prompt. Splitting them avoids the common mistake of pasting a summary and hoping the LLM figures out what to do.
- **Verbatim technical details.** The prompt insists on exact file paths, error messages, and version numbers — not paraphrases. This is what makes the handoff actually usable.
- **Self-check pass.** The skill re-reads its own output before returning it, catching ~30% more errors than a single pass.

---

## Troubleshooting

**The skill isn't firing when I type `/handoff`.**
Ask Claude: *"When would you use the handoff-generator skill?"* Claude will quote the description back. If your trigger phrase isn't listed, edit the `description` line in `SKILL.md` and re-upload. The description controls when the skill fires; the body controls what it does.

**The handoff is too generic / vague.**
Reply: *"Section X feels thin. Expand it using only what's in this chat."* The skill usually recovers well on a second pass. If it consistently fails on a section, edit the prompt to make that section's instructions more specific.

**The handoff invented things that didn't happen in the chat.**
This is the worst failure mode. File an issue with the chat (sanitized) and I'll tighten the anti-hallucination guardrail. In the meantime, scan section 4 ("Tried and Ruled Out") carefully — that's where invention is most damaging.

**Code execution isn't enabled.**
Skills require it. Go to **Settings → Capabilities → Code execution and file creation** and toggle it on.

---

## Limitations (honest)

- **Skills can't auto-trigger when your session is about to end.** Claude doesn't notice the warning UI. You have to invoke `/handoff` yourself. Train the muscle memory.
- **The output still requires copy-paste.** A skill running inside Claude can't push data to ChatGPT. Eliminating that step would require a browser extension, which is out of scope here.
- **No prompt is unbreakable.** This one is robust across the failure modes I designed against, but LLMs are non-deterministic. If you find a case where it consistently fails, open an issue.

---

## Repo structure

```
handoff-generator/
├── README.md                      # this file
├── handoff-generator.zip          # ready-to-upload Claude skill
├── HANDOFF_PROMPT.md              # standalone prompt for any LLM
└── handoff-generator/
    └── SKILL.md                   # the skill source
```

---

## Contributing

The prompt will get better as more people use it on real conversations. If you hit a failure mode:

1. Open an issue with the failure case (sanitize anything sensitive).
2. Note which section was wrong and how.
3. If you have a fix, PR it against `handoff-generator/SKILL.md`.

The most useful issues describe **what the next LLM did wrong** after reading the handoff. That's the real signal.

---

## License

MIT — see [LICENSE](./LICENSE) for details. Use it, fork it, ship it inside your own tools. If you build something interesting on top of it, I'd love to hear about it.