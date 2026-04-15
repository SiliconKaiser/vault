# Vault — Cloud Agent

## Overview

This repository combines a **framework** layer at the repository root (Cursor rules under `.cursor/`, skills, and this file) with **knowledge content** under `vault/`. Rules and skills describe behavior relative to **`vault/`** as the content root for topics, sources, templates, and the topic index.

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

For **`kb_interacting`** messages, assign exactly one intent:

1. **`query`** — Answer from stored knowledge only; **no** file changes under `vault/` for KB purposes.
2. **`update`** — Apply durable changes under `vault/` (topics, index, sources, logs as needed).
3. **`query_and_update`** — Perform **`update`** behavior first, then answer from the updated state. Log only what changed; do not log pure question text.

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

Framework material (`.cursor/`, this `AGENTS.md`) lives at the **repository root**; durable knowledge files live **under `vault/`**.

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

Use a short numeric or alphanumeric `<id>` unique for that calendar day. The year directory matches the date in the filename.

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

## Routing priority (KB updates)

When applying updates:

1. Prefer updating an **existing** topic when it fits.
2. Create a **new** topic only when no suitable topic exists.
3. If routing is ambiguous, ask **one** focused clarification in the chat reply before large or destructive edits.

## Skills (required)

- For **`kb_interacting`** messages with **`update`** or **`query_and_update`**: follow **`.cursor/skills/kb-topic-management/SKILL.md`** for routing, create/update/split, source files, and log appends.
- For **every** **`kb_interacting`** message (after KB handling): follow **`.cursor/skills/kb-response/SKILL.md`** to compose and deliver the short user-facing reply.

## Read-only vs mutating turns

- **`query`:** Do not modify topic files, `vault/topic-index.md`, or `vault/sources/`. Do not append log lines. Do not commit unless the user asked for something else that changes files.
- **`update` / `query_and_update`:** Update topics and `vault/topic-index.md` as needed; create source files; append logs; then **one git commit** with a short message and push to the default branch when vault content changed.

If unsure whether to persist, bias toward **mutating** when the user appears to want something remembered.

## Telegram reply

When the message arrived via Telegram (`[telegram_meta]` present), after work completes deliver the final user-visible text through the project’s Telegram integration (for example the Bot API `sendMessage` with `chat_id` from metadata). Store **`TELEGRAM_BOT_TOKEN`** (or equivalent) in Cursor **My Secrets** or agent environment, **never** in this repository.

Keep replies concise for mobile unless the user asks for detail.

## Git

Use a **single default branch** workflow unless the project owner specifies otherwise.

## Ambiguity and conflict

If the message is ambiguous or contradicts the vault, ask **one** focused clarification in the chat reply before making large or destructive edits. Prefer small, reversible updates when interpretation is unclear.
