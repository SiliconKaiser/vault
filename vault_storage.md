# Vault: Storage

Filesystem layout, lifecycle rules, root reference files, and file templates. Hub: [vault.md](vault.md). Behavior and skills: [vault_runtime.md](vault_runtime.md).

The tree below is **knowledge content**. Its root can be a different path or repository than the framework docs; `/vault` here means that root folder (name it however you like). Framework vs content: [vault.md](vault.md#framework-vs-knowledge-content).

---

## Directory layout

```
/vault
  /topics/               -- durable topics: active work (source of truth); see "Durable topics vs reference knowledge"
  /knowledge/
    glossary.md          -- reserved term lookup; see "Glossary" below (do not skip when scanning knowledge/)
    *.md                 -- other Markdown files here: reference knowledge (stable, not durable topics)
  /inbox/                -- inbox files awaiting processing (unprocessed)
  /inbox/notes.md        -- standing scratch file for quick capture (reset empty after inbox intake)
  /processed-inbox/      -- inbox files after processing is complete (summary at top, topics updated)
  /archived-topics/      -- obsolete topics only
  me.md                  -- personal context: role, team, company (stable reference)
  index.md               -- topic manifest, recent activity log
  docs.md                -- index of Google Docs (URLs, routing, summaries)
```

Five top-level directories (`topics`, `knowledge`, `inbox`, `processed-inbox`, and `archived-topics`). Three root files (`me.md`, `index.md`, `docs.md`).

### Durable topics vs reference knowledge

Two different kinds of Markdown, in two different places:

- **`/topics/`** holds **durable topics** (active work): projects, initiatives, tracked concerns. These use the topic file template (Current State, Decisions, Open Questions, Action Items, Key People, References). They change often and carry current state, decisions, active work, and references over time.
- **`/knowledge/*.md` except `glossary.md`** holds **reference knowledge**: how systems work, configuration, how-tos, comparisons. One file per subject, no required template, no durable-topic lifecycle. A durable topic may link to these files, but there is no 1:1 mapping. Do not call these "topics" in conversation when you mean durable topics; call them **reference knowledge files** (or **knowledge articles**) to avoid confusion with `/topics/`.

**Unprocessed vs processed**

- Everything in `/inbox/` **needs processing**, **except** `/inbox/notes.md` when it is empty or whitespace-only (standing capture slot).
- **`/inbox/`** -- inbox files awaiting processing. During processing, the AI adds a `## Summary` section to the top of the file, updates topics, then moves the file to `/processed-inbox/`. **`/inbox/notes.md`** -- default capture file; after any inbox file moves to `processed-inbox/`, `process-inbox` resets this file to blank for the next session.
- **`/archived-topics/`** -- durable topics that are obsolete; not part of the inbox pipeline.

### Glossary

**Path:** `knowledge/glossary.md`

**Why it exists:** `/knowledge/` can accumulate many reference files. The glossary is intentionally a **single, fixed-path** file so nothing replaces it by accident and so agents always know where to look for term definitions.

**What it is:** A flat, alphabetical table of acronyms, code names, jargon, and short definitions. It is **not** a durable topic and **not** a long-form reference article.

**Operational rule:** On orient and parse (see [vault_runtime.md](vault_runtime.md)), always read `knowledge/glossary.md` alongside `me.md`, `index.md`, and `docs.md`. Do not treat the glossary as "just another file in a crowded folder" -- it is the designated term index. When scanning `knowledge/` for domain context, read glossary first, then open other `knowledge/*.md` files as needed.

### What lives where


| Artifact               | Location                                          | Lifecycle                                                    |
| ---------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| Inbox file (unprocessed)| Conversation or `/inbox/`                       | AI adds summary, updates topics, then moves file to `/processed-inbox/`; `inbox/notes.md` is cleared |
| Inbox file (processed) | `/processed-inbox/`                               | Reference and provenance (summary at top, then original inbox content); linked from topics |
| Durable knowledge      | `/topics/`                                        | Accumulates over time, edited in place                       |
| Reference knowledge    | `/knowledge/*.md` except `glossary.md`            | Stable domain reference, edited when facts change            |
| Archived topic         | `/archived-topics/`                               | Moved when obsolete only                                     |
| Open items / questions | Inside the relevant topic file                    | Removed when resolved                                        |
| Action items           | Inside the relevant topic file                    | Updated in place; checked off or removed when done           |
| Message suggestions    | Conversation                                      | Ephemeral guidance during a session                          |
| Glossary terms         | `/knowledge/glossary.md`                          | Grows over time, rarely removed                              |
| Personal context       | `me.md`                                           | Updated when role, team, or company context changes          |
| Google Doc references  | `docs.md`                                         | Indexed on demand, summaries refreshed when stale            |


### Reference files

On startup, the AI reads `me.md`, `index.md`, `knowledge/glossary.md`, and `docs.md` to orient. It also reads other reference knowledge files under `knowledge/` as needed during processing.

**`me.md`** -- personal context: role, team, company, expertise, and communication preferences. Freeform sections, updated infrequently. The AI uses this to calibrate tone, understand org relationships, and avoid asking obvious questions.

**`knowledge/glossary.md`** -- see [Glossary](#glossary) above. Flat lookup table; alphabetical; consulted for unfamiliar terms during inbox processing; the AI proposes additions when it encounters new ones.

```markdown
# Glossary

| Term | Meaning |
|------|---------|
| DBRE | Database Reliability Engineering |
| SLI | Service Level Indicator |
```

**Other `knowledge/*.md` files** -- reference knowledge, one subject per file (for example how a system works, bucket config, or local dev setup). Not durable topics: no required topic template and no durable-topic lifecycle. Durable topics under `/topics/` may link to them; keep basenames distinct from durable topic basenames so `[[wiki-style]]` links resolve unambiguously (see [vault_runtime.md](vault_runtime.md#linking)).

**`docs.md`** -- index of Google Docs the AI should know about. Not a copy of the docs -- a routing layer so the AI knows what exists, what it covers, and when to fetch the live version via MCP.

Each entry has three parts:

```markdown
# Google Docs Index

## DNS Migration Design Doc
**URL**: https://docs.google.com/document/d/1abc.../edit
**Revision**: rev-abc123
**Last indexed**: 2026-03-18

### Routing
- Topics: [[dns-migration]], [[platform-infra]]
- Owner: @alice
- Type: design doc
- Status: approved

### Summary & Key Takeaways
- Proposes migrating from on-prem DNS to Route53
- Key constraint: must maintain <50ms resolution time during cutover
- Phased rollout: staging (March), canary (April), full (May)
- Rollback plan: keep old resolvers warm for 2 weeks post-cutover

## Caching Layer RFC
**URL**: https://docs.google.com/document/d/2def.../edit
**Revision**: rev-def456
**Last indexed**: 2026-03-15

### Routing
- Topics: [[caching-layer]]
- Owner: @bob
- Type: RFC
- Status: in review

### Summary & Key Takeaways
- Evaluates Redis vs Memcached for platform API caching
- Recommends Redis (persistence, data structures)
- Open question: cluster topology (single vs multi-region)
```

- **URL**: the doc's Google Docs URL (the permanent reference).
- **Revision**: revision ID from `get_document_metadata` MCP. Used to detect staleness.
- **Last indexed**: when the summary was last generated/refreshed.
- **Routing**: which topics relate, who owns it, what kind of doc, current status. This is what the AI reads to decide whether to fetch the full doc during a conversation.
- **Summary & Key Takeaways**: concise extraction of what matters. Enough to jog your memory and inform the AI's reasoning without fetching the live doc.

### Topic file structure

```markdown
# <Topic Name>

## Current State
What is true right now.

## Decisions
- YYYY-MM-DD: What was decided (source: [[summary-ref]])

## Open Questions
- Unresolved items

## Action Items
- [ ] Task (owner, urgency)

## Key People
- @person: role/relevance

## References
- [[summary-ref]]
```

Message suggestions, when useful, happen inline in the conversation. Topic files focus on durable state, decisions, open questions, action items, key people, and references.

### Summary section (at the top of inbox files)

After parsing an inbox item, the AI inserts this section at the top of the file (before the original inbox content):

```markdown
## Summary

**Date processed**: YYYY-MM-DD
**Topics**: [[topic-a]], [[topic-b]]

### Key Points
- ...

### Decisions
- ...

### Action Items
- [ ] Task (owner, urgency)
```

Action Items in a processed inbox summary are a snapshot of what the inbox item produced at processing time. If an item remains active after processing, carry it into the relevant topic's `Action Items` section so it can be updated there over time.

### index.md

```markdown
# Vault

## Topics
- [[topic-name]] - one-line description

## Recently Processed
- YYYY-MM-DD: what was processed -> what was updated
```

On startup, the AI reads `me.md`, `index.md`, `knowledge/glossary.md`, and `docs.md` to orient. The AI maintains `index.md` after each processing session. It proposes updates to `knowledge/glossary.md` when it encounters new terms, and to `me.md` when it learns new stable personal or org context. It proposes updates to `docs.md` when it encounters new Google Doc URLs or detects stale entries. It proposes new or updated reference knowledge files (additional `knowledge/*.md`, other than `glossary.md`) when long-form reference material emerges from processing. Topic References sections link to processed inbox files by basename (e.g. `[[2026-03-18-slack-dns-cutover]]` resolves to `processed-inbox/2026-03-18-slack-dns-cutover.md`).
