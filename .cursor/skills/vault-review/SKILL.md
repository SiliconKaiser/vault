---
name: vault-review
description: >-
  Performs a Vault review session: orient from index and root reference files,
  list unprocessed inbox items, scan topics for open questions, action items,
  and stale or blocked work, check Google Doc revision staleness via MCP, and surface actionable
  items. Use when the user asks what needs attention, for a vault status pass,
  or to triage backlog before processing.
---

# Vault: review

1. **Orient** -- Read `me.md`, `index.md`, `knowledge/glossary.md`, `docs.md` at the knowledge root.
2. **Scan backlog** -- List every file in `inbox/`.
3. **Scan topics** -- Collect open questions, action items, stale or blocked items, and any missing context that needs follow-up.
4. **Doc staleness** -- Compare `docs.md` revision IDs against MCP `get_document_metadata`. Flag mismatches.
5. **Surface** -- Present findings. Ask what to tackle.
6. **Act** -- Transition into the relevant skill per user choice.
