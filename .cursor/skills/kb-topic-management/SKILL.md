---
name: kb-topic-management
description: Manage life-admin topic files under vault/ — route, create, update, split topics; source records; append-only logs with provenance. Use when the user message is kb_interacting with update or query_and_update intent.
---

# Knowledge-base topic management

## When to invoke

- Any **`kb_interacting`** message classified as **`update`** or **`query_and_update`**.

## Inputs

- User message (full text)
- Message source metadata (e.g. Cursor chat, Telegram — for `source:` field in source records)
- `vault/topic-index.md`
- Candidate topic files under `vault/topics/` (by slug)
- Source ID context for the day: next free `YYYY-MM-DD-<source-id>` under `vault/sources/YYYY/` (increment `<source-id>` per new source file that day)

## Procedure

1. **Confirm intent** — Must be `update` or `query_and_update` (if only `query`, do not use this skill for edits).
2. **Route** — Prefer an existing topic whose subject matches; read `vault/topic-index.md` and relevant `vault/topics/<slug>.md` files. If no topic fits, plan a new `topic-slug` (lowercase kebab-case `[a-z0-9-]+`). If ambiguous, stop and return a single clarification question for the caller to resolve.
3. **Create topic if needed** — Copy `vault/templates/topic-template.md` to `vault/topics/<topic-slug>.md`, set the `#` title, add `- [[topic-slug]] — description` to `vault/topic-index.md`.
4. **Source record** — For substantive updates, create `vault/sources/YYYY/YYYY-MM-DD-<source-id>.md` with required fields: `source`, `timestamp` (ISO-8601), `message`, optional `notes`.
5. **Apply edits** — Write content to the correct sections only:
   - Slow-changing truth → `## Stable facts`
   - Current snapshot → `## Current State`
   - Tasks → `## Action Items`
   - Do not put historical narrative only in Current State when it belongs in `## Log`.
6. **Log** — For each meaningful change, append one line to `## Log` using the canonical format with a relative link to the source file. Skip log/source creation for pure queries (not applicable to this skill’s primary path except when bundled with `query_and_update` after updates).
7. **Action items** — Mark done/cancelled in `## Action Items`; log completion/cancellation when it is a meaningful change (with source link).
8. **Split** — If one topic mixes separable subjects, propose boundaries; after user confirmation if needed, create new topic(s), move content, update index and `[[wiki-links]]`, append logs on affected topics with source links.

## Output to caller

Return a concise machine-usable summary, for example:

- `intent`: `update` | `query_and_update`
- `touched_topics`: list of slugs
- `changes`: short bullet list of what changed per topic
- `source_files`: paths created under `vault/sources/` (if any)
- `needs_clarification`: boolean; if true, include exactly one question

The exact formatting can match the agent’s internal convention; keep it minimal and deterministic for the **`kb-response`** step.
