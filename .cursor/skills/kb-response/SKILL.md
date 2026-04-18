---
name: kb-response
description: Compose and deliver short KB-facing replies after knowledge-base handling — answer, changes, touched topics; channel-aware (Telegram vs chat). Use for every kb_interacting message after KB work.
---

# Knowledge-base response delivery

## When to invoke

- Any **`kb_interacting`** message **after** query handling and/or **`kb-topic-management`** completes.

Runs for **`query`**, **`update`**, and **`query_and_update`** so the user always gets a consistent, concise KB reply.

## Inputs

- Message source metadata (e.g. Telegram vs Cursor vs other)
- Answer text (from topic reads / reasoning)
- Update summary (from topic management; empty if query-only)
- Touched topic slugs (if any)

## Procedure

1. **Compose** one short user-facing message that includes:
   - The **answer** if the user asked something
   - **What changed** if anything was updated
   - **Which topic(s)** (`[[slug]]` or plain slug list) were read or modified
2. **Clarification** — At most **one** question, only if routing or intent is still unclear; otherwise omit.
3. **Deliver**:
   - If the task prompt includes **`[telegram_meta]`**, call the Telegram Bot API **`sendMessage`** (HTTPS POST to `https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/sendMessage` with JSON `chat_id` and `text`) using the token from Cursor **My Secrets**
   - Otherwise, output the final text as the normal chat assistant reply

## Constraints

- No verbose output: no long quoted excerpts from vault files unless the user explicitly asked for them
- At most one clarifying question
- Match tone to a quick status update, not a full report
