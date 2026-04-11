---
name: vault-quick-update
description: >-
  Records a user-stated fact, decision, status update, or action-item change into Vault topic files without running the
  full inbox workflow: clarify topic and change, propose a patch, optional
  message suggestions in chat. Use when the user reports a decision, status update, or
  fact to capture, or says quick-update in a vault context.
---

# Vault: quick-update

Read `index.md`, `me.md`, `knowledge/glossary.md`, and `docs.md` at the knowledge root.

1. **Understand** -- Identify topic and change. One change at a time.
2. **Update** -- Propose edits (template in `vault_storage.md`). Use the topic's `Action Items` section for mutable tasks and status changes. Apply after approval. Update `index.md` / `[[links]]` if needed. Run `vault-topic-split` and `vault-topic-archive`; only surface real suggestions.
3. **Follow-up communication** -- If anyone else needs to know, suggest wording in chat.

For long pasted threads or inbox files, use `vault-process-inbox` instead.
