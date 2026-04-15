---
name: kb-response
description: >-
  Compose concise KB user-facing replies and deliver via Telegram or normal chat after kb_interacting handling.
---

# KB response

## When to invoke

Every **`kb_interacting`** message **after** KB read/write work under `vault/` is complete (including **`query`**-only turns).

## Inputs

- Message source metadata (presence of `[telegram_meta]` and `chat_id`).
- Answer text (from vault for queries; from updated state for `query_and_update`).
- Update summary from kb-topic-management (or equivalent): touched topics, what changed, clarification flag.
- Optional: the single clarification question if routing was ambiguous.

## Procedure

1. **Compose** — Short, scannable message: answer (if any), vault changes (if any), topic slugs touched (if any), at most one clarification question only if needed.
2. **Channel** — If `[telegram_meta]` is present, deliver through the Telegram integration using `chat_id`. Otherwise return as the normal assistant message in chat.
3. **Deliver** — Send or return exactly one user-facing payload; avoid duplicate verbose system narration.

## Constraints

- No verbose dumps of file contents unless the user asks.
- At most **one** clarifying question; omit if not needed.
- Do not claim vault changes if none occurred on a `query` turn.
