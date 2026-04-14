---
name: vault-process-message
description: >-
  Handles the current user message against the Vault: classify read-only vs mutating;
  for reads, answer without editing files; for writes, update topics/knowledge/root files,
  add a records/ provenance file, update index.md, run topic-manage advisory checks,
  and make one git commit. Use for normal chat turns and any user content that should
  integrate into the vault.
---

# Vault: process-message

Read `me.md` and `index.md` at the knowledge root. Open `knowledge/*.md` as needed.

## 1. Classify

Determine whether fulfilling the message requires **changing** any vault files (`topics/`, `knowledge/`, `records/`, `me.md`, `index.md`).

- **Read-only:** Answering from existing files, search, or general reasoning without persisting new/changed vault state.
- **Mutating:** Any durable update, including new facts, decisions, action items, new topics, reference articles, or stable `me.md` / `index.md` updates.

If unclear, bias toward **mutating** when the user appears to want something remembered or recorded.

## 2. Read-only path

Answer in the chat reply. **Do not** create or edit vault files, **do not** add `records/`, **do not** commit.

## 3. Mutating path

1. **Integrate** -- Apply updates to the right topics and/or `knowledge/` and root files. Carry ongoing work into each relevant topic's **Action Items** where appropriate. No approval gate. On ambiguity or conflict, use `AskQuestion` (one multiple-choice question at a time).
2. **Record** -- Create **one** new file under `records/` using the template in `vault_storage.md`. Include the user message verbatim or faithfully quoted, list every path changed, and summarize knowledge impact. Skip this step only if classification was read-only (then you should not be on this path).
3. **Index** -- Update `index.md` (topics list and/or recent activity) to reflect changes.
4. **Topic lifecycle** -- Run advisory **split** evaluation from `vault-topic-manage` on touched topics; surface only substantive suggestions.
5. **Follow-up** -- If useful, suggest outbound messages in the chat reply.
6. **Git** -- Create **one commit** containing all file changes from this message; use a short descriptive message. If the workspace is not a git repo, skip.

## 4. Reply

Give a concise chat response: what you did or found, names of topics touched (for mutating), or the direct answer (for read-only).
