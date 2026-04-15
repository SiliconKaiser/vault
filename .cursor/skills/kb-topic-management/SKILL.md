---
name: kb-topic-management
description: >-
  Route, create, update, and split topics under vault/; manage action items;
  create source records and append logs for KB updates and query_and_update flows.
---

# KB topic management

## When to invoke

Any **`kb_interacting`** message classified as **`update`** or **`query_and_update`**.

## Inputs

- User message text and classification (`update` vs `query_and_update`).
- Message source metadata (e.g. Telegram vs Cursor chat) for the `source` field in source records.
- `vault/topic-index.md` for routing.
- Candidate topic files under `vault/topics/` opened or inferred from the message.
- Calendar context for `vault/sources/YYYY/YYYY-MM-DD-<id>.md`: choose `<id>` unique for that day.

## Procedure

1. **Confirm intent** — Treat as mutating KB work; do not create sources or logs for read-only paths mixed into the same turn incorrectly.
2. **Route** — Prefer an existing `vault/topics/<topic-slug>.md`. If none fit, plan a new canonical slug (`[a-z0-9-]+`). If ambiguous, stop after preparing **one** clarification question (caller delivers via kb-response).
3. **Create topic** (if needed) — Copy structure from `vault/templates/topic-template.md` into `vault/topics/<topic-slug>.md`; add `- [[topic-slug]] — …` to `vault/topic-index.md`.
4. **Update sections** — Stable truth → `## Stable facts`; volatile now → `## Current State`; tasks → `## Action Items`; meaningful history → append `## Log` with source link.
5. **Action items** — Add, edit, complete, or cancel items in `## Action Items`. On completion or cancellation that materially changes state, append `## Log` with source link when appropriate.
6. **Split** (if warranted) — After substantive edits, if the file mixes separable subjects: propose moves; confirm with user if boundaries are unclear; then create/move/update index and `[[links]]`; log substantive moves with source links.
7. **Source record** — For each discrete change event that changes stored knowledge, create **one** `vault/sources/YYYY/YYYY-MM-DD-<id>.md` with required fields (`source`, `timestamp`, `message`, optional `notes`).
8. **Log append** — For each meaningful change, append one line under `## Log` using the canonical pattern (relative to the topic file):

   `- YYYY-MM-DD — <summary>. Source: [<stem>](../sources/YYYY/YYYY-MM-DD-<id>.md)`

9. **Git** — If `vault/` content changed, one commit and push per agent run unless the user specifies otherwise.

## Output to caller

Return a concise machine-usable summary for downstream steps (for example kb-response), including:

- `intent`: `update` | `query_and_update`
- `topics_touched`: list of slugs
- `changes`: short bullet list of what changed (sections, new topics, splits)
- `source_files`: list of created `YYYY-MM-DD-<id>.md` stems (if any)
- `needs_clarification`: boolean; if true, include the single question text
