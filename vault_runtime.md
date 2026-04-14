# Vault: Runtime

Conversation model, agent skills, topic maintenance, workflows, and linking. Hub: [vault.md](vault.md). Layout and templates: [vault_storage.md](vault_storage.md).

---

## Interaction model

Processing is **message-based**: each user message is handled in one pass. The AI classifies the message as **read-only** (question, retrieval, clarification without edits) or **mutating** (anything that should change stored knowledge).

- **Read-only:** Answer using the vault and tools. Do **not** create or edit vault files, do **not** add a record under `records/`, and do **not** make a vault commit for that message.
- **Mutating:** Integrate the message into `topics/`, `knowledge/`, and root files as appropriate; create a **record** under `records/` with the user message and list of changes; update `index.md`; run **topic-manage** checks; finish with **one git commit** for that message that includes every file touched (see [Git](#git)).

There is no inbox backlog folder. Optional capture outside this spec (local scratch files) is not part of the vault pipeline.

### Principles

- **Autonomous by default.** For mutating messages, the AI parses, updates files, writes the record, and finalizes unless ambiguity or conflict requires a stop.
- **Questions only on ambiguity or conflict.** Use `AskQuestion` with succinct multiple-choice options. One question at a time. No open-ended prompts.
- **Chat-first.** Replies should stand alone: summarize what was done or found, point to topic titles by name, and avoid assuming the user sees the repository tree in an editor.
- **Message suggestions in chat.** When follow-up with other people is useful, suggest wording in the user-facing reply.

---

## Git

When the environment is a Git repository, **each mutating user message** results in **exactly one commit** containing all vault edits from handling that message (topics, `knowledge/`, `records/`, `me.md`, `index.md`, link fixes, etc.). Read-only turns produce **no commit**.

Use a short, descriptive commit message (e.g. `vault: note Redis decision under caching topic`).

---

## Agent skills

Each skill is a distinct entry point. Execution steps live in `.cursor/skills/vault-<name>/SKILL.md`.

### 1. `process-message`

Primary skill. Handles a **user message** end-to-end: classify read-only vs mutating; for mutating work, integrate into the vault, add a `records/` file, update `index.md`, run `topic-manage` advisory steps, and commit once. Entry point is always the current user message (and any attached context), not a folder of pending files.

### 2. `review`

Start-of-session or on-demand. Orients from `me.md` and `index.md`, scans topics for open questions and action items, optionally skims recent `records/` for continuity, and surfaces what needs attention.

### 3. `topic-manage`

Topic lifecycle: **create** and **update** durable topics (template in `vault_storage.md`), **split** when one file mixes separable concerns. Use when editing topics or integrating new knowledge that affects topics—not only as a follow-up to `process-message`. Advisory split checks surface suggestions; the user decides in chat unless policy says otherwise.

---

## Workflows

| Situation                         | Skill             | Typical trigger                                      |
| --------------------------------- | ----------------- | ---------------------------------------------------- |
| User sent a message               | `process-message` | Default for normal chat turns                        |
| User wants orientation or triage  | `review`          | "What needs attention?", status pass                 |
| Editing topics or merging knowledge | `topic-manage`  | Structural topic work, split review                  |

The AI infers the skill from context; naming the skill explicitly is optional.

After any **mutating** `process-message` run, execute the **advisory** portions of `topic-manage` (split evaluation) on touched topics; surface only substantive suggestions.

---

## Chat-first use

When the user does not keep the vault open in an IDE:

- **Orient from files:** Treat `me.md` and `index.md` as the primary map; mention topic names and short descriptions in replies.
- **Self-contained answers:** For queries, synthesize from topic and reference files; cite which areas of the vault support the answer (by topic title or file role) without requiring the user to open paths.
- **Explicit completion:** For mutating turns, briefly confirm what was updated and name affected topics so the user can refer to them later in chat.
- **No hidden state:** Anything durable belongs in the vault files; the chat does not replace `topics/` or `records/`.

---

## Linking

Use wiki-style `[[name]]` links (matching the filename without extension). These are grep-friendly and concise.

**Convention:** Keep basenames unique across `topics/`, `knowledge/`, and `records/` so `[[name]]` resolves to exactly one file.

Examples:

- `[[dns-migration]]` links to `topics/dns-migration.md`
- `[[s3-bucket-config]]` links to `knowledge/s3-bucket-config.md`
- `[[2026-04-12-redis-decision]]` links to `records/2026-04-12-redis-decision.md`
