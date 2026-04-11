---
name: vault-index-doc
description: >-
  Indexes a Google Doc into the vault docs.md file: fetch metadata and content
  via MCP, build routing and summary sections, write after user confirmation.
  Use when the user provides a Google Docs URL to index, asks to refresh a
  stale docs.md entry, or when a new Doc URL appears during processing and the
  user agrees to index it.
---

# Vault: index-doc

Entry format: `vault_storage.md` (Google Docs Index section).

1. **Metadata** -- Call `get_document_metadata` via MCP for title and revision ID.
2. **Content** -- Call `get_document_content`. Extract images if MCP supports it.
3. **Draft entry** -- Build URL, revision, date, routing, summary. Confirm routing with user.
4. **Write** -- Append or replace the entry in `docs.md` at the knowledge root.

On re-index (revision differs from stored value): notify user, offer to refresh, flag affected topics.
