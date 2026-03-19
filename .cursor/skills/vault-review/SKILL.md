---
name: vault-review
description: >-
  Performs a Vault review session: orient from index and root reference files,
  list unprocessed queue and summaries, scan topics for open questions and
  pending outbound, check Google Doc revision staleness via MCP, and surface
  actionable items. Use when the user asks what needs attention, for a vault
  status pass, or to triage backlog before processing.
---

# Vault: review

## Before you start

1. Read `vault_runtime.md` from the framework root (directory with `vault.md`).
2. Locate the **knowledge root** (`topics/`, `knowledge/`, `index.md`).

## Steps

1. **Orient** -- Read `me.md`, `index.md`, `knowledge/glossary.md` (required; see `vault_storage.md` Glossary), `docs.md` at the knowledge root.
2. **Scan backlog** -- List every file in `queue/` and `summaries/` excluding `processed/` subtrees. Omit `queue/notes.md` when it is empty or whitespace-only (it is the standing capture file, not backlog until it has content). Offer each listed item as optional work (summarize queue items, integrate summaries into topics).
3. **Scan topics** -- Collect open questions, pending outbound, and anything that looks stale or blocked.
4. **Doc staleness** -- For entries in `docs.md`, when MCP is available call `get_document_metadata` (or equivalent) and compare revision IDs to stored values. Flag mismatches.
5. **Surface** -- Present backlog, topic attention items, stale docs, and pending outbound. Ask what to tackle.
6. **Act** -- For each user choice, continue with `vault-process-input`, `vault-quick-update`, `vault-index-doc`, or targeted edits as appropriate.

## Style

Questions, not a long unsolicited briefing. Keep items scannable.
