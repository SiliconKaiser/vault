# Vault — Cloud Agent

## Overview

This repository separates **framework** (how knowledge is managed) from **knowledge content** under `vault/`. Durable state lives in Markdown under `vault/` in a fixed layout. The user’s normal interface is **chat** (for example Telegram); they do not rely on an editor to read or update files.

**Your role:** For each task, you receive the user’s message (and optional routing metadata). You interpret it against the knowledge base, **read and edit files under `vault/`** when the user’s intent requires persisting or changing knowledge, **commit and push** when you change the repository, and **send a concise reply** on the messaging channel when applicable. You are the only writer: answers must be grounded in vault files and tools, not invented file paths.

**Ingress:** An external service may start a Cursor Cloud Agent run on this repo for each inbound chat message. The task prompt may begin with routing metadata, then the user’s text.

Rules and skills live at the **repository root** under `.cursor/`; topic pages, sources, and the topic index live under **`vault/`**.

## Objective

Maintain **durable life-admin and work knowledge** under `vault/topics/`. Each topic file is one coherent subject; **all four required sections** live in that file in a fixed order (see [Knowledge layout](#knowledge-layout)).

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

1. **`kb_interacting`** — The message participates in the knowledge-base flow (read and/or update stored topics).
2. **`non_kb_interacting`** — The message does not participate in the knowledge-base flow; respond with normal assistant behavior and **do not** run KB update/query procedures on `vault/topics/`, `vault/sources/`, or `vault/topic-index.md`.

For **`kb_interacting`** messages, assign exactly one intent:

1. **`query`** — Answer from stored knowledge only; **no** file changes under `vault/` for KB purposes.
2. **`update`** — Apply durable changes to topic(s), provenance, and logs as needed under `vault/`.
3. **`query_and_update`** — Perform **`update`** behavior first, then answer the question from the updated state. Log only what changed; do not log pure question text.

### Processing summary

| Intent | Topic files | Source records | Log lines |
|--------|-------------|----------------|-----------|
| `query` | Read only | Do not create | Do not append |
| `update` | Edit/create | Create when knowledge changes | Append for meaningful changes |
| `query_and_update` | Edit/create as needed | Create when knowledge changes | Append for meaningful changes only |

## Knowledge layout

All paths below are under the **`vault/`** content root.

| Path | Purpose |
|------|---------|
| `vault/topics/` | Flat list of topic files (`<topic-slug>.md`). Each file holds one subject. |
| `vault/topic-index.md` | Authoritative list of topics for routing (`[[topic-slug]]` entries). |
| `vault/sources/YYYY/` | Provenance: **one new file per update** that changes stored knowledge (see [Source records](#source-records)). |
| `vault/templates/topic-template.md` | Starting structure for new topics. |
| `vault/me.md` | Optional stable personal context (tone, preferences). |

Orient from `vault/me.md` and `vault/topic-index.md` first; open `vault/topics/*.md` as needed.

## Topic files

Each topic MUST live at `vault/topics/<topic-slug>.md`.

**Topic slugs** MUST be lowercase kebab-case: `[a-z0-9-]+` (examples: `car-maintenance`, `personal-finance`).

**Required headings** MUST appear in this **exact order**, each at most once as a level-2 heading:

1. `## Stable facts`
2. `## Current State`
3. `## Action Items`
4. `## Log`

Cross-topic links MUST use wiki-link form with the canonical slug: `[[topic-slug]]`.

### Placement

- Slow-changing truth → `## Stable facts`
- Volatile “what is true now” → `## Current State`
- Open commitments / tasks → `## Action Items`
- Meaningful dated change → append under `## Log` (append-only; see [Log](#log))

## Source records

When an **`update`** or **`query_and_update`** changes stored knowledge, create **exactly one** new file per such change event under:

`vault/sources/YYYY/YYYY-MM-DD-<id>.md`

Use a short numeric or alphanumeric `<id>` unique for that calendar day (for example `001`, `002`). The year directory matches the date in the filename.

**Minimum front matter / body fields** (adapt the shape as long as the fields exist):

```markdown
# Source Record: YYYY-MM-DD-<id>

- source: <origin description>
- timestamp: <ISO-8601 timestamp>
- message: "<user message content>"
- notes: <optional>
```

Examples of `source` values: `Telegram from user xyz`, `Cursor chat from user xyz`, `Manual note from user xyz`.

**Do not** create source files for **`query`**-only turns.

## Log

The `## Log` section is **append-only**. Each new log line MUST link to the source file that motivated the change.

Canonical line shape (paths relative to the topic file under `vault/topics/`):

```text
- YYYY-MM-DD — <event summary>. Source: [<stem>](../sources/YYYY/YYYY-MM-DD-<id>.md)
```

Use `<stem>` equal to the source filename without `.md` (for example `2026-04-15-001`).

**Do not** append log lines for **`query`**-only turns.

## Routing priority (KB updates)

When applying updates:

1. Prefer updating an **existing** topic when it fits.
2. Create a **new** topic only when no suitable topic exists.
3. If routing is ambiguous, ask **one** clarification question (in the user-facing reply) before large or destructive edits.

### Topic creation

1. Choose a canonical `topic-slug`.
2. Create `vault/topics/<topic-slug>.md` from `vault/templates/topic-template.md`.
3. Add a line to `vault/topic-index.md`: `- [[topic-slug]] — short description`
4. Write content into the correct section(s).
5. For substantive updates, append to `## Log` with a source link.

### Topic split

If one topic mixes two or more separable subjects after substantive edits: propose what moves and what stays; confirm with the user if boundaries are unclear; then create new topic file(s), move content, update `vault/topic-index.md` and `[[links]]`, and append log entries with source links on affected topics.

### Action items

Keep action items in `## Action Items` until done or cancelled. Completing or cancelling an item SHOULD produce a log line with a source link when it represents a meaningful state change.

## Skills (required)

- For **`kb_interacting`** messages with **`update`** or **`query_and_update`**: follow **`.cursor/skills/kb-topic-management/SKILL.md`** for routing, create/update/split, source files, and log appends.
- For **every** **`kb_interacting`** message (after KB handling): follow **`.cursor/skills/kb-response/SKILL.md`** to compose and deliver the short user-facing reply.

## Read-only vs mutating turns

- **`query` / read-only:** Do not modify topic files, `vault/topic-index.md`, or `vault/sources/`. Do not append log lines. Do not commit unless the user asked for something else that changes files.
- **`update` / `query_and_update`:** Update topics and `vault/topic-index.md` as needed; create source files; append logs; then **one git commit** with a short message and push to `main` when the vault content changed.

If unsure whether to persist, bias toward **mutating** when the user appears to want something remembered.

## Telegram reply

When the message arrived via Telegram (`[telegram_meta]` present), after work completes call the Telegram Bot API `sendMessage` with `chat_id` from the metadata. Store **`TELEGRAM_BOT_TOKEN`** in Cursor **My Secrets** (or agent environment), **never** in this repository.

Keep replies concise for mobile unless the user asks for detail.

## Git

Use a **single branch** workflow on `main` unless the project owner specifies otherwise.

## Ambiguity and conflict

If the message is ambiguous or contradicts the vault, ask **one** focused clarification in the chat reply before making large or destructive edits. Prefer small, reversible updates when interpretation is unclear.
