---
name: vault-process-input
description: >-
  Runs the Vault process-input pipeline on raw or queued input: parse, confirm,
  resolve conflicts, assess relevance, write summaries, update topics with user
  approval, outbound drafts, and queue/summary file moves. Use when the user
  pastes Slack, Jira, meeting notes, or general notes; points at a file in
  queue/ or an unprocessed summary in summaries/; or asks to process vault
  input or drain the backlog.
---

# Vault: process-input

## Before you start

1. Framework detail: read `vault_runtime.md` from the **framework root** (directory that contains `vault.md` and `.cursor/`).
2. Find the **knowledge root** (directory containing `topics/` and `knowledge/`). If multiple workspace folders exist, prefer the one with `topics/` and `index.md`.
3. Read `me.md`, `index.md`, `knowledge/glossary.md`, and `docs.md` at the knowledge root when processing input.

## Pipeline (gate each write on user approval)

Execute in order; skip trailing steps if the user ends early (see runtime for low-stakes exceptions).

1. **Parse** -- Classify source type. Read `knowledge/glossary.md` first for terms; use `me.md` and other `knowledge/*.md` reference files as needed. See `vault_storage.md` (glossary section).
2. **Confirm understanding** -- State what you extracted; ask for corrections. Propose new glossary terms if needed.
3. **Resolve conflicts** -- If input contradicts a topic, show both sides; ask what to keep or mark as open question.
4. **Assess relevance** -- Use `index.md`; ask what matters.
5. **Write summary** -- New file under `summaries/` (or edit in place if session started there). If source was in `queue/`, move that file to `queue/processed/` once the summary exists. **After any move from `queue/` to `queue/processed/`**, ensure `queue/` exists at the knowledge root, then create or overwrite `queue/notes.md` with empty content (no text; optional single trailing newline only) so the next capture slot is ready.
6. **Update topics** -- Propose edits; apply only after approval. Then move summary from `summaries/` to `summaries/processed/`. Run **topic-split** and **topic-archive** checks (`vault_runtime.md`).
7. **Generate outbound** -- If needed, ask goal/tone; draft in conversation; user sends manually.
8. **Record outcomes** -- Update topic **Outbound Log** with what was communicated.

## Templates

Summary and topic shapes: `vault_storage.md`.

## Google Doc URLs

If you find a new Doc URL, ask whether to run **index-doc** (`vault-index-doc` skill).
