# Vault repository

This GitHub repository holds the **knowledge framework** (`AGENTS.md`, `.cursor/skills/`, specification files) and **durable content** under `vault/` (topics, sources, topic index). Layout and agent rules are defined in `life-admin-knowledge-spec.md` and `AGENTS.md`.

## Cursor Cloud Agents

Agents need access to clone and push this repository. In [Cursor](https://cursor.com): **Dashboard → Cloud Agents → Connect Git**, and grant access to this repo.

How runs are started (chat ingress, API, pull requests, merges) is configured in **your operator tooling and GitHub settings**, not in this repository.

## `.github/workflows/`

Optional GitHub Actions in this repository run on GitHub only. What each workflow does is defined in the workflow file.
