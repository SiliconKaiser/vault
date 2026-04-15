# Vault — Cloud Agent

## Overview

This repository combines a **framework** layer at the repository root (this file, `.cursor/skills/`) with **knowledge content** under `vault/`. **Agent policy for the knowledge base lives in this document** so it stays portable; path behavior is expressed relative to **`vault/`** as the content root for topics, sources, templates, and the topic index.

The user’s typical interface is **chat** (for example Telegram); they do not rely on an editor to read or update files.

**Your role:** For each task, interpret the user’s message against the knowledge base, **read and edit files** under `vault/` when intent requires persisting or changing knowledge, **commit and push** when you change repository content, and **send a concise reply** on the messaging channel when applicable.

## Objective

Maintain **durable life-admin and work knowledge** under `vault/topics/`, with provenance under `vault/sources/`, routing via `vault/topic-index.md`, and new topics from `vault/templates/topic-template.md`.

## Task prompt shape (when present)

```text
[telegram_meta]
chat_id=...
message_id=...
username=...
[/telegram_meta]

<user message>
```

Parse `chat_id` (and optionally `message_id`, `username`) from the block. Ignore the block for vault content; it is only for replies.

## Message classification

Classify every incoming user message into exactly one top-level type:

1. **`kb_interacting`** — The message participates in the knowledge-base flow (read and/or update stored topics under `vault/`).
2. **`non_kb_interacting`** — The message does not participate; respond with normal assistant behavior and **do not** run KB update/query procedures on `vault/topics/`, `vault/sources/`, or `vault/topic-index.md`.

For **`kb_interacting`** messages, assign **exactly one** intent:

1. **`query`** — Answer from stored knowledge only; **no** file changes under `vault/` for KB purposes.
2. **`update`** — Apply durable changes under `vault/` (topics, index, sources, logs as needed).
3. **`query_and_update`** — Perform **`update`** behavior first, then answer from the updated state. Log only what changed; do not log pure question text.

Do **not** assign multiple intent paths to one message. If both Q&A and persistence apply, use **`query_and_update`**.

### Processing summary

| Intent | Topic files under `vault/topics/` | Source records under `vault/sources/` | Log lines |
|--------|-----------------------------------|----------------------------------------|------------|
| `query` | Read only | Do not create | Do not append |
| `update` | Edit/create | Create when knowledge changes | Append for meaningful changes |
| `query_and_update` | Edit/create as needed | Create when knowledge changes | Append for meaningful changes only |

## Vault layout (paths relative to `vault/`)

| Path | Purpose |
|------|---------|
| `topics/` | Topic files: `vault/topics/<topic-slug>.md`. Each file holds one subject. |
| `topic-index.md` | Authoritative list of topics for routing (`[[topic-slug]]` entries). |
| `sources/YYYY/` | Provenance: one new file per update that changes stored knowledge. |
| `templates/topic-template.md` | Starting structure for new topics. |
| `me.md` | Optional stable personal context (tone, preferences). |

Framework material (this file, `.cursor/skills/`) lives at the **repository root**; durable knowledge files live **under `vault/`**.

## Topic files

Each topic MUST live at `vault/topics/<topic-slug>.md`.

**Topic slugs** MUST be lowercase kebab-case: `[a-z0-9-]+` (examples: `car-maintenance`, `personal-finance`).

**Required headings** MUST appear in this **exact order**, each at most once as a level-2 heading:

1. `## Stable facts`
2. `## Current State`
3. `## Action Items`
4. `## Log`

Cross-topic links MUST use wiki-link form with the canonical slug: `[[topic-slug]]`.

### Placement by knowledge type

| Kind of information | Section |
|---------------------|---------|
| Slow-changing reference truth | `## Stable facts` |
| Volatile “true now” | `## Current State` |
| Open tasks / commitments | `## Action Items` |
| Meaningful dated history | `## Log` (append-only) |

### Routing priority

When applying updates:

1. Prefer updating an **existing** topic when it fits the subject.
2. Create a **new** topic only when no suitable topic exists.
3. If multiple topics could apply and the right choice matters, ask **one** focused clarification question before large edits.

### Topic creation

1. Choose a canonical `topic-slug`.
2. Create `vault/topics/<topic-slug>.md` from `vault/templates/topic-template.md`.
3. Add `- [[topic-slug]] — description` to `vault/topic-index.md`.
4. Write content into the correct section(s).
5. For substantive updates, append to `## Log` with a source link.

### Topic split

When one file mixes **separable** subjects: define what moves and what stays; confirm with the user if unclear; create new topic file(s) under `vault/topics/`; move content; update `vault/topic-index.md` and `[[wiki-links]]` elsewhere; append `## Log` lines with source links where substantive.

### Action items

Items stay under `## Action Items` until completed or cancelled. Completion or cancellation that materially changes state SHOULD add a `## Log` line with a source link.

### Split prompting

Do not re-prompt split on every tiny edit—only when the topic meaningfully changed or the user asks.

## Source records

When an **`update`** or **`query_and_update`** changes stored knowledge, create **exactly one** new file per such change event under:

`vault/sources/YYYY/YYYY-MM-DD-<id>.md`

`<id>` is unique per calendar day. The year directory matches the date in the filename.

**Minimum fields** (adapt shape as long as the fields exist):

```markdown
# Source Record: YYYY-MM-DD-<id>

- source: <origin description>
- timestamp: <ISO-8601 timestamp>
- message: "<user message content>"
- notes: <optional>
```

**Do not** create source files for **`query`**-only turns.

## Log

The `## Log` section is **append-only**. Each new log line MUST link to the source file that motivated the change.

Paths in log links are relative to the topic file under `vault/topics/`:

```text
- YYYY-MM-DD — <event summary>. Source: [<stem>](../sources/YYYY/YYYY-MM-DD-<id>.md)
```

Use `<stem>` equal to the source filename without `.md` (for example `2026-04-15-001`).

**Do not** append log lines for **`query`**-only turns.

For **`update`** and **`query_and_update`**, when stored knowledge changes, create a source record for that change event and link it from each new log line that records a meaningful change.

## Skills (required)

- For **`kb_interacting`** messages with **`update`** or **`query_and_update`**: follow **`.cursor/skills/kb-topic-management/SKILL.md`** for routing, create/update/split, source files, and log appends.
- For **every** **`kb_interacting`** message (after KB handling): follow **`.cursor/skills/kb-response/SKILL.md`** to compose and deliver the short user-facing reply.

## Read-only vs mutating turns

- **`query`:** Do not modify topic files, `vault/topic-index.md`, or `vault/sources/`. Do not append log lines. Do not commit unless the user asked for something else that changes files.
- **`update` / `query_and_update`:** Update topics and `vault/topic-index.md` as needed; create source files; append logs; then **one git commit** with a short message and push to the default branch when vault content changed.

If unsure whether to persist, bias toward **mutating** when the user appears to want something remembered.

## KB response delivery

For every **`kb_interacting`** message, after KB handling (read and/or write under `vault/`), produce a **short, easy-to-read** user-facing message.

**Include only what is essential:**

- Direct answer to the question (if any).
- What changed in the vault (if anything).
- Which topic slug(s) were touched (if any).
- At most **one** clarification question, and only when needed.

Avoid long explanations unless the user asks for detail.

**Channel:**

- **Telegram** (`[telegram_meta]` present): deliver the final user-visible text through the project’s Telegram integration (for example Bot API `sendMessage` with `chat_id` from metadata). Store **`TELEGRAM_BOT_TOKEN`** (or equivalent) in Cursor **My Secrets** or agent environment, **never** in this repository. Keep replies concise for mobile unless the user asks for detail.
- **Other chat:** return the final text as the normal assistant reply.

The full procedure is in **`.cursor/skills/kb-response/SKILL.md`**.

## Git

Use a **single default branch** workflow unless the project owner specifies otherwise.

## Ambiguity and conflict

If the message is ambiguous or contradicts the vault, ask **one** focused clarification in the chat reply before making large or destructive edits. Prefer small, reversible updates when interpretation is unclear.
