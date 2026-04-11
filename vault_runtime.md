# Vault: Runtime

Conversation model, agent skills, topic maintenance, doc indexing, workflows, and linking. Hub: [vault.md](vault.md). Layout and templates: [vault_storage.md](vault_storage.md).

---

## Interaction Model

The system is **conversation-driven, not event-driven.** You initiate by providing content. The AI leads a Q&A conversation to process it. No batch jobs, no background automation, no scheduled pipelines.

### How it works

1. You provide raw content (paste content or point to a `/inbox/` file).
2. AI parses it, adds a summary section at the top of the inbox file, updates topics, and handles file moves -- all autonomously.
3. If follow-up communication would help, AI can suggest wording inline in the conversation. You send manually.
4. AI finalizes: moves processed inbox file and records topic updates.

### Principles

- **Autonomous by default.** The AI parses, summarizes, updates topics, and finalizes without asking. It proceeds end-to-end unless it hits ambiguity or conflict.
- **Questions only on ambiguity or conflict.** The AI asks a question only when (a) the content could reasonably be interpreted multiple ways, or (b) new information contradicts something already recorded. All other decisions the AI makes on its own.
- **Structured questions.** When the AI does ask, it uses the `AskQuestion` tool to present a succinct multiple-choice question. No open-ended "what do you think?" prompts. One question at a time.
- **You can steer at any point.** The conversation is not a fixed script. Corrections after the fact are fine -- the AI adjusts.
- **AI adapts depth to stakes.** Routine inbox work: fast, no questions. Sensitive or contradictory: pause and ask.
- **Message suggestions stay in conversation.** The AI may suggest wording inline, while topic files stay focused on durable knowledge.

---

## Agent Skills

Each skill is a distinct entry point -- a different reason you start a conversation with the AI. Execution steps for each skill live in `.cursor/skills/vault-<name>/SKILL.md`.

### 1. `process-inbox`

The primary skill. Takes raw content through the full inbox pipeline: parse, summarize, update topics and action items, and suggest follow-up communication in chat when useful. Entry points: pasted content or a file in `/inbox/`. Anything in `/inbox/` (excluding empty `notes.md`) is fair game.

### 2. `review`

Start-of-session or on-demand. Scans the vault and surfaces what needs attention: inbox backlog, open questions, action items, stale docs, blocked items.

### 3. `quick-update`

Record a decision, fact, or action-item change directly into a topic -- no inbox item to process. You tell the AI something; it updates the right topic.

### 4. `glossary-detect`

Detects domain-specific or org-internal terms missing from `knowledge/glossary.md`. Asks for a definition via `AskQuestion`, then appends. Runs automatically inside `process-inbox` and `quick-update` (after parsing), or standalone when the user uses an unfamiliar term in conversation.

### 5. `topic-split` / `topic-archive`

Not standalone workflows -- they run automatically as part of `process-inbox` and `quick-update` after any topic update. Advisory: the AI surfaces observations and the user decides.

---

## Bookkeeping

### `index-doc`

Index a Google Doc into `docs.md`. Triggered explicitly ("index this doc" + URL) or detected during `process-inbox` when a new Doc URL appears. Also handles staleness: `review` compares stored revision IDs against current via MCP and offers to re-index changed docs.

---

## Workflows

All workflows are **user-initiated and conversation-driven.** No background processes, no cron jobs, no event triggers. You start every workflow by either providing content or asking a question.

Each workflow maps to one of the three core skills:

| Trigger                      | Skill           | Example                                           |
| ---------------------------- | --------------- | ------------------------------------------------- |
| You provide raw content      | `process-inbox` | "Here's a Slack thread about the DNS cutover"     |
| You ask what's going on      | `review`        | "What needs my attention?"                        |
| You state a fact or decision | `quick-update`  | "We decided to use Redis for the caching layer"   |

The AI determines which skill applies from context. You never need to name the skill explicitly.

Glossary detection (`glossary-detect`) runs automatically within `process-inbox` and `quick-update` after parsing, or standalone. Topic management (`topic-split`, `topic-archive`) runs automatically within `process-inbox` and `quick-update` when topics are updated. Bookkeeping skills like `index-doc` are invoked on demand or detected during processing.

**Backlog rule:** Files in `/inbox/` are always treated as needing work, except `/inbox/notes.md` when empty or whitespace-only. `review` surfaces them; `process-inbox` is the primary way to drain them.

---

## Linking

Use wiki-style `[[name]]` links (matching the filename without extension). These are grep-friendly and concise.

Resolution depends on your editor or tooling. **Convention:** keep basenames unique across `topics/` and `knowledge/` so `[[name]]` points to exactly one file. The glossary is only `knowledge/glossary.md` and is not typically linked via `[[glossary]]` unless you standardize that.

Examples:

- `[[dns-migration]]` links to `topics/dns-migration.md` (a durable topic)
- `[[s3-bucket-config]]` links to `knowledge/s3-bucket-config.md` (reference knowledge; not a durable topic)
- `[[2026-03-18-slack-dns-cutover]]` links to an inbox file by basename -- in `/inbox/` while unprocessed, or `/processed-inbox/` after processing (same basename; resolves by filename if unique)
