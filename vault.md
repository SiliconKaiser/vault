# Vault

A local-first system for turning noisy inputs into durable knowledge and tracked action -- powered by AI conversation.

## Core Idea

AI processes information. You make decisions. Knowledge accumulates in topic files. Messages are ephemeral; outcomes are recorded.

```
Input  -->  Signal  -->  Knowledge  -->  Action
(you)       (AI)        (AI + you)      (AI + you)
```

## Framework vs knowledge content

**Framework** is this specification: the Markdown hub files ([vault_storage.md](vault_storage.md), [vault_runtime.md](vault_runtime.md), and this page) plus Cursor integration in this repo: `.cursor/rules/` (always-on and path-specific behavior), `.cursor/skills/` (per-skill `SKILL.md` files aligned with runtime), and `.cursor/commands/` (slash commands that dispatch those workflows). **Knowledge content** is the on-disk tree described in storage: `topics/`, `knowledge/`, `queue/`, `summaries/`, `archive/`, and the root files such as `index.md` and `me.md`.

Those two can live in different folders or Git repos. A typical layout is a small framework checkout for the spec and a separate directory for the vault tree; open both in one multi-root workspace or link the tree so agents and editors can read the spec and edit content together.

---

## Documentation map

| Doc | What it covers |
| --- | -------------- |
| [vault_storage.md](vault_storage.md) | `/vault` tree, durable topics vs reference knowledge, glossary note, lifecycle table, `me.md` / `knowledge/` / `docs.md`, topic and summary templates, `index.md` |
| [vault_runtime.md](vault_runtime.md) | Conversation model, `process-input` / `review` / `quick-update`, topic-split and topic-archive, `index-doc`, workflows, `[[linking]]` |

## Read order (for agents)

1. This file (hub and principles).
2. [vault_storage.md](vault_storage.md) (where files live and what they look like).
3. [vault_runtime.md](vault_runtime.md) (how to run skills and conversations).

Point Cursor rules or onboarding at the hub plus runtime if behavior matters most; add storage when editing layout or templates.

---

## Workflows (quick reference)

All workflows are user-initiated and conversation-driven. No background automation.

| Trigger                      | Skill           | Example                                           |
| ---------------------------- | --------------- | ------------------------------------------------- |
| You provide raw content      | `process-input` | "Here's a Slack thread about the DNS cutover"     |
| You ask what's going on      | `review`        | "What needs my attention?"                        |
| You state a fact or decision | `quick-update`  | "We decided to use Redis for the caching layer"   |

**Backlog rule:** Files in `/queue/` and `/summaries/` (excluding `processed/` subtrees) need work, except `/queue/notes.md` when it is empty or whitespace-only (standing capture file). `review` surfaces them; `process-input` is the primary way to drain them. Details: [vault_runtime.md](vault_runtime.md#workflows).

---

## Operating Principles

1. **Conversation is the interface.** Files are where decisions land, not where work happens.
2. **AI processes, you decide.** The AI never commits knowledge or sends messages without your say.
3. **Durable topics** (files under `topics/`) are the only durable work artifact. Reference knowledge under `knowledge/` and everything else are supporting material; see [vault_storage.md](vault_storage.md).
4. **Track outcomes, not drafts.** Record what was communicated, not the draft of what might be.
5. **Start lean.** Add structure only when you feel pain from its absence.
