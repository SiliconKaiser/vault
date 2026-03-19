---
name: vault-quick-update
description: >-
  Records a user-stated fact or decision into Vault topic files without full
  input processing: clarify topic and change, propose a patch, optional
  outbound message. Use when the user reports a decision, status update, or
  fact to capture, or says quick-update in a vault context.
---

# Vault: quick-update

## Before you start

1. Framework reference: `vault_runtime.md` (framework root).
2. **Knowledge root** with `topics/`, `knowledge/`, and `index.md`.

## Steps

1. **Understand** -- Ask which topic (or create one if user agrees) and what changed. One change at a time if multiple.
2. **Update** -- Propose concrete edits to the topic file matching the template in `vault_storage.md`. Apply only after user approval. Update `index.md` / `[[links]]` if needed. Then run **topic-split** and **topic-archive** checks (`vault_runtime.md`).
3. **Outbound** -- Ask if anyone else needs to know; if yes, draft a message in conversation and record the outcome in **Outbound Log** after the user sends it.

Do not use this skill for long pasted threads; use `vault-process-input` instead.
