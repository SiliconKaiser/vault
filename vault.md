# Vault

A system for turning user messages into durable knowledge and tracked action—primarily through a chat interface, with markdown in a filesystem-backed vault.

## Core idea

```
User message  -->  AI  -->  Knowledge + records (when changed)
(chat)              |         (markdown on disk)
                    |
                    +-->  Reply only (read-only query)
```

## Framework vs knowledge content

**Framework** is this specification: the Markdown hub files ([vault_storage.md](vault_storage.md), [vault_runtime.md](vault_runtime.md), and this page) plus Cursor integration in this repo: `.cursor/rules/` (always-on and path-specific behavior), `.cursor/skills/` (per-skill `SKILL.md` files aligned with runtime), and `.cursor/commands/` (slash commands that dispatch those workflows, if present). **Knowledge content** is the on-disk tree described in storage: `topics/`, `knowledge/`, `records/`, and root files such as `index.md` and `me.md`.

Those two can live in different folders or Git repos. A typical layout is a small framework checkout for the spec and a separate directory for the vault tree; open both in one multi-root workspace or link the tree so agents and editors can read the spec and edit content together.

---

## Documentation map

| Doc | What it covers |
| --- | -------------- |
| [vault_storage.md](vault_storage.md) | `/vault` tree, durable topics vs reference knowledge, `records/`, `me.md` / `knowledge/`, topic and record templates, `index.md` |
| [vault_runtime.md](vault_runtime.md) | Message model, `process-message` / `review` / `topic-manage`, read-only vs mutating, git commit rule, `[[linking]]`, chat-first behavior |
| [life_admin_knowledge_abstract.md](life_admin_knowledge_abstract.md) | Abstract model for personal life admin: kinds of truth, raw input as explanation, history vs overwrite, fit with the vault tree |

## Read order (for agents)

1. This file (hub and principles).
2. [vault_storage.md](vault_storage.md) (where files live and what they look like).
3. [vault_runtime.md](vault_runtime.md) (how to run skills and handle each message).

Point Cursor rules or onboarding at the hub plus runtime if behavior matters most; add storage when editing layout or templates.

---

## Workflows (quick reference)

| Trigger | Skill | Example |
| ------- | ----- | ------- |
| User message | `process-message` | "Remember we picked Redis for cache" |
| Status / triage | `review` | "What needs my attention?" |
| Topic structure work | `topic-manage` | Splitting a topic or merging content |

Details: [vault_runtime.md](vault_runtime.md#workflows).

---

## Operating principles

1. **Chat is the default interface; files are durable.** Replies live in the conversation; knowledge lives in the vault.
2. **AI acts; the user steers.** Update topic and reference files autonomously. Pause only for genuine ambiguity or conflicting information.
3. **Durable topics** (under `topics/`) are the main work artifact. Reference knowledge under `knowledge/` supports them; see [vault_storage.md](vault_storage.md).
4. **Mutating messages leave a trail.** A new file under `records/` captures the user message and what changed. Read-only queries do not.
5. **One commit per mutating message** when using Git. See [vault_runtime.md](vault_runtime.md#git).
6. **Start lean.** Add structure only when pain appears.
