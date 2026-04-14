---
name: vault-topic-manage
description: >-
  Topic lifecycle for the Vault: create and update durable topics (template in vault_storage),
  evaluate split when one file mixes separable concerns.
  Use when editing topics or integrating new knowledge; also runs as advisory checks after
  mutating process-message turns.
---

# Vault: topic-manage

Read `index.md` and `vault_storage.md` for templates and layout.

## Create

When a new concern needs durable tracking, add `topics/<basename>.md` using the topic template. Ensure the basename is unique across `topics/`, `knowledge/`, and `records/`. Update `index.md` and add `[[links]]` from related topics or records.

## Update

Edit topics in place: Current State, Decisions, Open Questions, Action Items, Key People, References. Prefer linking to `records/` for provenance when the change came from a mutating message.

## Split (advisory)

After substantive edits, evaluate: does this topic clearly contain **two or more separable** subjects?

- If yes: describe what would become its own topic and what would stay. Ask whether to proceed (`AskQuestion` if user choice is required). If approved: create the new file, move content, update `index.md`, fix `[[links]]` in other topics and records.
- If no or user declines: leave as-is.

Do not re-prompt split on every tiny edit; only when the topic was meaningfully changed or the user asks.
