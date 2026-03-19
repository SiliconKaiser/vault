# Vault: Runtime

Conversation model, agent skills, topic maintenance, doc indexing, workflows, and linking. Hub: [vault.md](vault.md). Layout and templates: [vault_storage.md](vault_storage.md).

---

## Interaction Model

The system is **conversation-driven, not event-driven.** You initiate by providing input. The AI leads a Q&A conversation to process it. No batch jobs, no background automation, no scheduled pipelines.

### How it works

1. You provide raw input (paste content or point to a `/queue/` file).
2. AI reads it, states its understanding, and asks you to confirm or correct.
3. AI asks small, sequential questions to build context -- one judgment call at a time.
4. Based on your answers, AI writes or updates summaries and topics. Queue source files move to `/queue/processed/` after summarization (then `/queue/notes.md` is reset empty as the next capture slot); summary files move to `/summaries/processed/` after topic integration.
5. If outbound messages are needed, AI generates them inline. You send them manually.
6. AI records outcomes in the relevant topic file.

### Principles

- **AI leads with questions, not reports.** No "here's my briefing" documents. The AI asks and you answer.
- **Questions are small and sequential.** Not "here are 6 decisions." One at a time, each building on the last.
- **You can steer at any point.** The conversation is not a fixed script.
- **AI adapts depth to stakes.** Routine: fewer questions, faster. Sensitive: more questions, more back-and-forth.
- **AI proposes, you commit.** AI never writes to topic files without your approval during the conversation.

### Per input type

**Slack thread**: AI reads -> confirms understanding -> asks what matters -> updates topic -> asks if you need to reply -> generates message -> records outcome.

**Jira ticket**: AI reads -> summarizes state -> asks what changed -> updates topic -> asks if messages are needed -> generates them -> records outcome.

**Meeting notes**: AI reads -> extracts decisions -> asks which are firm vs tentative -> updates topics -> asks about follow-ups -> generates messages if needed.

**General note**: AI reads -> asks what you're working through -> helps refine thinking -> asks if anything should become durable knowledge.

---

## Agent Skills

Each skill is a distinct entry point -- a different reason you start a conversation with the AI.

### 1. `process-input`

The primary skill. Takes raw input through the full pipeline: extract signal, build understanding via Q&A, write summary, update topics, generate outbound messages. One skill, multiple steps, all in a single conversation.

**Entry points:** pasted content, a file in `/queue/`, or an existing unprocessed file in `/summaries/` (integrate into topics, then move to `/summaries/processed/`). Anything in those two unprocessed directories is fair game for this skill.

Steps (in order, each gated by user approval):

1. **Parse** -- read raw input, identify source type (Slack, Jira, meeting, note). Consult `knowledge/glossary.md` first for unfamiliar terms (see [vault_storage.md](vault_storage.md#glossary)). Use `me.md` and other `knowledge/*.md` reference files as needed for org and domain context.
2. **Confirm understanding** -- state what the AI extracted, ask user to confirm or correct. Propose new glossary entries for any unrecognized terms.
3. **Resolve conflicts** -- if the input contradicts existing topic knowledge, surface the contradiction explicitly. Present both the existing state and the new information, and ask the user which is correct or whether to record it as an open question.
4. **Assess relevance** -- identify related topics (consulting `index.md`), ask user what matters
5. **Write summary** -- create structured summary as a new file in `/summaries/` (unprocessed), unless the session started from an existing file in `/summaries/` (then edit in place). If the input came from `/queue/`, move that source file to `/queue/processed/` once the summary exists, then create or overwrite `/queue/notes.md` with empty content so the next capture file is ready.
6. **Update topics** -- propose changes to relevant topic files, apply with user approval. After updates are accepted, move the summary file from `/summaries/` to `/summaries/processed/`. Then run `topic-split` and `topic-archive` checks (see Topic Management below).
7. **Generate outbound** -- if messages are needed, ask about goal/tone, draft inline
8. **Record outcomes** -- log what was communicated in the topic's Outbound Log

Not every input reaches every step. A low-stakes note might stop after step 5 with a summary still in `/summaries/` (topics not updated yet). A critical Slack thread might complete the full pipeline in one session.

**Stages:** (1) queue item -> summary written -> `/queue/processed/` -> blank `/queue/notes.md`; (2) summary integrated into topics -> `/summaries/processed/`. If there is no queue file (pasted input), only stage (2) applies to the summary file (no `notes.md` reset unless a queue move occurred).

### 2. `review`

Start-of-session or on-demand. Scans the vault and tells you what needs attention.

Steps:

1. **Orient** -- read `me.md`, `index.md`, `knowledge/glossary.md` (always; see [vault_storage.md](vault_storage.md#glossary)), `docs.md`, understand current state
2. **Scan backlog** -- list every file in `/queue/` and `/summaries/` (not in `processed/` subdirectories). Skip `/queue/notes.md` when it is empty or whitespace-only. Offer each remaining item as work to pick up (summarize queue items, integrate pending summaries into topics).
3. **Scan topics** -- read topic files, collect open questions, pending outbound, stale items
4. **Check doc staleness** -- for indexed docs, call `get_document_metadata` and compare revision IDs. Flag any docs that have changed since last indexed.
5. **Surface** -- present backlog (queue + unprocessed summaries), topic attention items, stale docs, pending outbound; ask user what to act on
6. **Act** -- for each item the user picks, transition into the relevant action (`process-input` for a queue file or summary, update topic, re-index doc, generate message, etc.)

### 3. `quick-update`

For when you have a decision or fact to record and there's no input to process. You tell the AI something; it updates the right topic.

Steps:

1. **Understand** -- ask which topic and what changed
2. **Update** -- propose change to topic file, apply with approval. After updating, run `topic-split` and `topic-archive` checks (see Topic Management below).
3. **Outbound** -- ask if anyone needs to know, generate message if yes

---

## Topic Management

These are not standalone skills -- they run automatically as part of `process-input` and `quick-update` whenever a topic is updated. They are advisory: the AI surfaces observations and the user decides what to do.

### `topic-split`

After updating a topic, the AI evaluates whether the topic file has grown to cover more than one distinct concern. If the AI believes a split would help, it asks the user.

Behavior:

- The AI does **not** apply hard rules or line-count thresholds. It uses judgment: does this file clearly contain two or more separable subjects?
- If yes, the AI describes the proposed split (what would become its own topic, what would stay) and asks the user whether to proceed.
- If the user agrees, the AI creates the new topic file, moves the relevant content, updates `index.md`, and fixes any `[[links]]` in summaries and other topics.
- If the user declines, the AI does nothing. It does not ask again until the next time the topic is updated.

### `topic-archive`

After updating a topic, the AI evaluates whether the topic is ready to be archived -- obsolete or no longer relevant.

Behavior:

- The AI looks at the topic's current state: is the subject obsolete? Are open questions resolved or moot? Has the topic gone stale with no recent activity?
- If the AI believes the topic is a candidate for archiving, it tells the user why and asks whether to archive it.
- If the user agrees, the AI moves the file from `/topics/` to `/archive/topics/`, updates `index.md` to reflect the change, and notes the archive date.
- If the user declines, the AI does nothing. It does not ask again until the next time the topic is updated.

---

## Bookkeeping

Utility skills that maintain the system's supporting artifacts. These are not core workflows -- they keep the plumbing working.

### `index-doc`

Index a Google Doc into `docs.md`. Triggered two ways:

- **Explicitly**: you say "index this doc" and provide a URL. AI proceeds directly.
- **Detected**: during `process-input`, the AI encounters a Google Doc URL not already in `docs.md`. AI asks if you want to index it.

Steps:

1. **Fetch metadata** -- call `get_document_metadata` via MCP to get title, revision ID
2. **Fetch content** -- call `get_document_content` via MCP to read the full doc
3. **Generate entry** -- produce the three-part index entry (URL + revision, routing, summary + key takeaways). Ask user to confirm or adjust routing info (related topics, owner, status).
4. **Write** -- append the entry to `docs.md`

**Staleness detection**: during `review` or any time the AI consults `docs.md`, it can call `get_document_metadata` to compare the stored revision ID against the current one. If they differ, the doc has changed and the AI should:

1. Notify the user that the doc has been updated since last indexed
2. Offer to re-index (fetch content, regenerate summary, update revision ID)
3. Flag if the changes might affect any related topics

---

## Workflows

All workflows are **user-initiated and conversation-driven.** No background processes, no cron jobs, no event triggers. You start every workflow by either providing input or asking a question.

Each workflow maps to one of the three core skills:

| Trigger                      | Skill           | Example                                           |
| ---------------------------- | --------------- | ------------------------------------------------- |
| You provide raw content      | `process-input` | "Here's a Slack thread about the DNS cutover"     |
| You ask what's going on      | `review`        | "What needs my attention?"                        |
| You state a fact or decision | `quick-update`  | "We decided to use Redis for the caching layer"   |

The AI determines which skill applies from context. You never need to name the skill explicitly.

Topic management (`topic-split`, `topic-archive`) runs automatically within `process-input` and `quick-update` when topics are updated. Bookkeeping skills like `index-doc` are invoked on demand or detected during processing.

**Backlog rule:** Files in `/queue/` and `/summaries/` (excluding `processed/` subtrees) are always treated as needing work, except `/queue/notes.md` when empty or whitespace-only. `review` surfaces them; `process-input` is the primary way to drain them.

---

## Linking

Use wiki-style `[[name]]` links (matching the filename without extension). These are grep-friendly and concise.

Resolution depends on your editor or tooling. **Convention:** keep basenames unique across `topics/` and `knowledge/` so `[[name]]` points to exactly one file. The glossary is only `knowledge/glossary.md` and is not typically linked via `[[glossary]]` unless you standardize that.

Examples:

- `[[dns-migration]]` links to `topics/dns-migration.md` (a durable topic)
- `[[s3-bucket-config]]` links to `knowledge/s3-bucket-config.md` (reference knowledge; not a durable topic)
- `[[2026-03-18-slack-dns-cutover]]` links to `/summaries/2026-03-18-slack-dns-cutover.md` while unprocessed, or `/summaries/processed/2026-03-18-slack-dns-cutover.md` after integration (same basename; Foam resolves by filename if unique)
