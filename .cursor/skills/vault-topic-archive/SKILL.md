---
name: vault-topic-archive
description: >-
  Evaluate whether a topic is obsolete or stale and propose archiving it.
  Runs as a check after topic updates in process-inbox or quick-update,
  or standalone if the user asks.
---

# Vault: topic-archive

Evaluate the updated topic file. Is the subject obsolete? Are open
questions resolved or moot? Has the topic gone stale with no recent
activity?

- If yes: explain why and ask the user whether to archive.
- If approved: move from `topics/` to `archived-topics/`, update
  `index.md`, note the archive date.
- If declined: do nothing. Do not ask again until the topic is next
  updated.
