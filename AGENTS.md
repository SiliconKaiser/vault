# Agent policy: life-admin knowledge base

This repository contains a **framework** (rules and skills at the repo root) and **knowledge content** under `vault/`. All paths below are relative to the repository root unless stated otherwise.

## Roles

- **Framework** (`AGENTS.md`, `.cursor/skills/`) — Shareable instructions for *how* to manage knowledge. It does not contain personal data.
- **Knowledge content** (`vault/`) — Durable topic pages, provenance sources, templates, and the topic index. Agents read and write here.

`AGENTS.md` and skills live at the repo root; durable files for topics and sources live under `vault/`. Do not duplicate this policy in separate Cursor rule files (for example `.mdc` under `.cursor/rules/`); this file is the single source of truth for these rules.

## Objectives

- Maintain durable life-admin and work knowledge in wiki-style topic pages under `vault/topics/`.
- Prefer updating existing topics before creating new ones.
- Preserve provenance: substantive updates create source records; log entries link to those sources.
- After any knowledge-base handling for KB-interacting messages, compose the user-facing reply using the **`kb-response`** skill.

## Message classification

Every incoming user message is classified into exactly one top-level type:

1. **`kb_interacting`** — The message participates in the knowledge-base flow (queries and/or updates to stored knowledge).
2. **`non_kb_interacting`** — The message does not participate; respond as a normal assistant without KB routing, topic edits, source records, or KB-style log appends.

For **`kb_interacting`** messages, assign exactly one intent:

1. **`query`** — Read stored knowledge and answer; do not modify topics.
2. **`update`** — Apply changes to topics (and provenance) as needed.
3. **`query_and_update`** — Apply updates first, then answer from the updated state.

## Topic pages

### Location and index

- Topic files: `vault/topics/<topic-slug>.md`
- Authoritative list for routing: `vault/topic-index.md`

### Topic slug

- Lowercase kebab-case: `[a-z0-9-]+` (e.g. `car-maintenance`, `personal-finance`, `mom-health`).

### Required headings (exact order)

Every topic page **must** contain these four sections, in this order:

1. `## Stable facts`
2. `## Current State`
3. `## Action Items`
4. `## Log`

### Section roles

- **Stable facts** — Slow-changing reference truths for this topic (corrected or updated when reality changes).
- **Current State** — What is true now and expected to change; overwrite as things move.
- **Action Items** — Open commitments until done or cancelled; optional due/urgency.
- **Log** — Dated, append-only lines when meaningful changes occur; every entry links to its source record (see Provenance).

### Cross-topic links

Use wiki links with canonical slugs only:

- `[[topic-slug]]`

## Processing rules by intent

### Query

- Read relevant files under `vault/topics/` (and `vault/topic-index.md` for routing).
- Return the answer.
- Do **not** modify topic files, append log entries, or create source records.

### Update

- Route to existing topic(s) or create a topic if none fits.
- Apply updates to the correct section(s).
- Create a source record under `vault/sources/` (see Provenance).
- Append a log entry for each meaningful change, with a link to the source file.

### Query and update

- Perform **update** behavior first, then answer the query from the updated state.
- Log only substantive changes; do not log pure question text as if it were a knowledge change.

### Non-KB interacting

- Do not run the KB update/query flow for topic files.
- Handle the message with normal assistant behavior.

## Topic management

Use the **`kb-topic-management`** skill for any `kb_interacting` message with intent **`update`** or **`query_and_update`**.

### Routing priority

1. Prefer updating an existing topic when it fits.
2. Create a new topic only when no suitable topic exists.
3. If routing is ambiguous, ask **one** clarification question.

### Placement

- Stable truth → `## Stable facts`
- Volatile “now” state → `## Current State`
- Commitments/tasks → `## Action Items`
- Meaningful dated change → append to `## Log` (with source link)

### Creating a topic

1. Choose a canonical `topic-slug`.
2. Create `vault/topics/<topic-slug>.md` from `vault/templates/topic-template.md`.
3. Add a line to `vault/topic-index.md`: `- [[topic-slug]] — short description`
4. Fill the correct section(s) with initial content.
5. For substantive updates, append log entries with source links.

### Action items

- Keep open items in `## Action Items` until done or cancelled per user/project policy.
- Completing or cancelling an item should produce a log entry with a source link when it is a meaningful state change.

### Splitting a topic

When one file mixes two or more separable subjects:

1. Decide what moves to a new topic and what stays.
2. Confirm with the user if boundaries are unclear.
3. Create new topic file(s), move content, fix `[[links]]`.
4. Update `vault/topic-index.md`.
5. Append log entries with source links on affected topics for substantive moves.

Do not re-prompt splits on every tiny edit—only when the topic meaningfully changed or the user asks.

## Provenance and logging

### Source records

- Store all source files under: `vault/sources/YYYY/YYYY-MM-DD-<source-id>.md`
- `<source-id>` is unique per calendar day (e.g. `001`, `002`).

Each source file **must** include at least:

```md
# Source Record: YYYY-MM-DD-<source-id>

- source: <origin description>
- timestamp: <ISO-8601 timestamp>
- message: "<user message content>"
- notes: <optional>
```

Examples of `source` values: `Telegram from user …`, `Cursor chat from user …`, `Manual note from user …`.

### Log entries (in topic files)

- Append only; do not rewrite historical log lines except to fix clear errors with user direction.
- Every log line **must** reference the source file that triggered the change.

Canonical format (paths relative to `vault/topics/<topic-slug>.md`):

```text
- YYYY-MM-DD — <event summary>. Source: [<source-id>](../sources/YYYY/YYYY-MM-DD-<source-id>.md)
```

Pure **query** KB messages **must not** create source files or log entries.

## Response delivery

Use the **`kb-response`** skill for **every** `kb_interacting` message after KB handling completes.

### Channels

- If the message originated from **Telegram**, send the final user response through the Telegram command interface (the send command is implemented outside this policy).
- Otherwise, return the response in normal chat.

### Content

Keep KB responses short and scannable:

- Answer (if any)
- What changed (if any)
- Which topic(s) were touched
- At most **one** clarification question, only when needed

Avoid verbose dumps; no more than one clarifying question.

## Skills (invocation summary)

| Skill | When |
| ----- | ---- |
| `kb-topic-management` | `kb_interacting` + (`update` or `query_and_update`) |
| `kb-response` | Any `kb_interacting` message after KB handling |

Classification and routing use `vault/topic-index.md` and candidate topics under `vault/topics/`.

## Git (Cursor Cloud Agent)

Runs started by **Cursor Cloud Agents** for this repository (for example from CloudVault / Telegram) use a direct-push workflow. After any edits under `vault/`, create **one** commit with a short message and **push directly to `main`** on `origin` (no PR workflow).
