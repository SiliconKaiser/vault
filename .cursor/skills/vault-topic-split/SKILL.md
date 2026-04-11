---
name: vault-topic-split
description: >-
  Evaluate whether a topic file covers more than one distinct concern and
  propose splitting it. Runs as a check after topic updates in process-inbox
  or quick-update, or standalone if the user asks.
---

# Vault: topic-split

Evaluate the updated topic file. Does it clearly contain two or more
separable subjects?

- No hard line-count thresholds -- use judgment.
- If yes: describe what would become its own topic and what would stay.
  Ask the user whether to proceed.
- If approved: create the new topic file, move the relevant content,
  update `index.md`, and fix any `[[links]]` in other topics and
  processed inbox files.
- If declined: do nothing. Do not ask again until the topic is next
  updated.
