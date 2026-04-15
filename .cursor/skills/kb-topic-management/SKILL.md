---
name: kb-topic-management
description: >-
  Route, create, update, and split topics under vault/; manage action items;
  create source records and append logs for KB updates and query_and_update flows.
---

# KB topic management

## When to invoke

Any **`kb_interacting`** message classified as **`update`** or **`query_and_update`**.

Do **not** use this skill as the primary guide for **`query`**-only messages (those do not mutate topics or provenance).

## Inputs

- User message text.
- Message source metadata (for example Telegram vs Cursor chat) when available.
- `vault/topic-index.md` (authoritative topic list).
- Candidate topics under `vault/topics/` (filenames and slugs match).
- Existing source files under `vault/sources/YYYY/` for the current day (to pick the next `<id>`).

## Procedure

1. **Intent** — Confirm the turn is **`update`** or **`query_and_update`** (not **`query`** alone).
2. **Route** — Map content to one or more existing topic slugs, or decide a new slug is needed. Prefer existing topics. If ambiguous, stop and surface **one** clarification question via the response skill.
3. **Create** — If creating a topic: copy `vault/templates/topic-template.md` to `vault/topics/<topic-slug>.md`, add the index line to `vault/topic-index.md`, then fill sections.
4. **Update** — Edit `## Stable facts`, `## Current State`, and/or `## Action Items` as appropriate. Maintain heading order.
5. **Action items** — Add, complete, or cancel items in `## Action Items`; add a `## Log` line when completion/cancellation is a meaningful change.
6. **Source record** — For substantive knowledge changes, add `vault/sources/YYYY/YYYY-MM-DD-<id>.md` with required fields.
7. **Log** — For each meaningful change, append one line to `## Log` with date, summary, and link to the source file per `AGENTS.md`.
8. **Split** — If the topic should be split, follow the split steps in `.cursor/rules/01-topic-management-core.mdc` and log in each affected topic.

## Output to caller

Return a concise machine-usable summary for downstream steps, for example:

- `touched_slugs`: list of topic slugs edited or created.
- `changes`: one-line descriptions per topic or global.
- `source_stems`: list of source filenames without `.md` if any were created.
- `needs_clarification`: boolean; if true, include `clarification_prompt`.

The response skill uses this to build the user-facing message.
