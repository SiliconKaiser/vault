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

## Before you start

1. Entry format and fields: `vault_storage.md` (Google Docs Index section).
2. Staleness and re-index flow: `vault_runtime.md` (index-doc and review).

## Steps

1. **Metadata** -- Call `get_document_metadata` (or equivalent MCP) for title and revision ID.
2. **Content** -- Call `get_document_content` to read the doc body.
3. **Draft entry** -- Build URL, revision, last indexed date, routing (topics, owner, type, status), and summary / key takeaways. Ask the user to confirm or adjust routing.
4. **Write** -- Append or replace the entry in `docs.md` at the **knowledge root**.

## Images

If the MCP stack includes image extraction for Docs, follow product guidance to capture important figures (see any workspace Google Workspace server instructions).

## Re-index

If revision differs from stored value, notify the user, offer to re-fetch content and update revision and summary, and note if related topics may need review.
