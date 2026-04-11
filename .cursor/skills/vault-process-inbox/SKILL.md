---
name: vault-process-inbox
description: >-
  Runs the Vault process-inbox workflow on raw content or an inbox file: parse, write
  summary, update topics and action items, suggest follow-up communication in chat. AI runs autonomously and only
  involves the user at genuine decision points.
---

# Vault: process-inbox

Read `index.md`, `me.md`, `knowledge/glossary.md`, and `docs.md` at the knowledge root.

1. **Parse** -- Classify source, extract signal. Consult glossary and reference files.
2. **Summarize** -- Add a `## Summary` at the top of the inbox file (template in `vault_storage.md`). For pasted content, create a dated inbox file.
3. **Update topics** -- Apply changes directly. Carry ongoing work into each relevant topic's `Action Items` section so it can be tracked there after the inbox file is processed. No approval gate. If anything is ambiguous (e.g. unclear which topic is relevant, unclear interpretation) or conflicts with other info, use the `AskQuestion` with a succinct multiple-choice question before proceeding, one question at a time.
4. **Follow-up communication** -- If useful, suggest messages to send to others or mention that a Jira ticket should be updated. Keep that guidance in the conversation. If unclear whether follow-up communication is warranted, ask via `AskQuestion`.
5. **Finalize** -- Move inbox file to `processed-inbox/`. Reset `inbox/notes.md` to empty. Update `index.md`. Run `vault-topic-split` and `vault-topic-archive`; only surface real suggestions.
