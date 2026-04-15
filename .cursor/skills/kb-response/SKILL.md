---
name: kb-response
description: >-
  Compose concise KB user-facing replies and deliver via Telegram or normal
  chat after kb_interacting handling.
---

# KB response

## When to invoke

Any **`kb_interacting`** message **after** KB handling completes (whether the intent was **`query`**, **`update`**, or **`query_and_update`**).

## Inputs

- Message source metadata (presence of `[telegram_meta]`, channel label).
- Answer text (from reading topics for **`query`**; from updated state for writes).
- Update summary from topic management (touched slugs, what changed), or empty for pure **`query`**.
- Optional: one clarification question if routing required it.

## Procedure

1. **Compose** a short message containing:
   - The answer (if there was a question).
   - What changed and which `[[topic-slug]]` names were touched (if any).
   - At most one clarification question, only if still required.
2. **Choose delivery channel:**
   - If Telegram metadata is present: use the Telegram Bot API `sendMessage` with `chat_id` from metadata (implementation of credentials is environment-specific).
   - Otherwise: respond in normal chat with the same text.
3. **Deliver** the final user-visible string through that channel.

## Constraints

- No verbose dumps of file contents unless the user explicitly asks.
- At most **one** clarifying question per reply.
- Mobile-friendly brevity for Telegram-style channels.
