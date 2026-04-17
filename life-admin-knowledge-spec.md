# Life Admin Knowledge System - Implementation Specification

## Purpose

This document defines the file layout, behavior rules, skill boundaries, and processing flow for a life-admin knowledge system managed by an LLM agent.

The specification is implementation-facing and self-contained.

## Repository roles: framework vs vault

This system uses a **single repository** with two layers:

1. **Framework (repository root, excluding `vault/`)** — Shareable material that describes *how* knowledge management works: root **`AGENTS.md`** (complete agent policy for the knowledge base, including all behavioral rules—message classification, topic management, provenance, and response delivery), **`.cursor/skills/`**, and optional supporting documents (including this specification and the abstract model). Others can copy or fork the repo and use this layer as a template without inheriting anyone’s personal knowledge.

2. **Knowledge content (`vault/`)** — The durable personal knowledge tree: topic pages, provenance sources, templates, and the topic index. This directory is the **content root** for all paths that refer to `topics/`, `sources/`, `templates/`, and `topic-index.md`.

Agents and documentation MUST treat paths as relative to **`vault/`** unless stated otherwise. **`AGENTS.md`** (all rules and policy) and skills under **`.cursor/skills/`** live at the repo root; they instruct the agent to read and write files under `vault/`. Do not rely on separate Cursor rule files (for example `.mdc` under `.cursor/rules/`); the single source of truth for rules is **`AGENTS.md`**.

## Repository Layout

```text
<repo-root>/                          # vault-framework: framework + optional vault content
  AGENTS.md                           # full agent policy: all rules + KB classification, vault paths, provenance, responses
  life-admin-knowledge-spec.md        # this document (or equivalent supporting doc)
  life_admin_knowledge_abstract.md    # abstract model (optional but recommended)
  .cursor/
    skills/
      kb-topic-management/
        SKILL.md
      kb-response/
        SKILL.md
  vault/                              # actual knowledge — omit or replace when sharing only the framework
    topic-index.md
    topics/
      <topic-slug>.md
    sources/
      YYYY/
        YYYY-MM-DD-<source-id>.md
    templates/
      topic-template.md
    me.md                               # optional stable personal context (if used)
```

## Core Concepts

### Topic Page

A topic page is the durable unit of knowledge.

Each topic page MUST exist at `vault/topics/<topic-slug>.md` and MUST contain these headings in this exact order:

1. `## Stable facts`
2. `## Current State`
3. `## Action Items`
4. `## Log`

### Four Knowledge Sections

- **Stable facts**: slow-changing truths for the topic.
- **Current State**: what is true now and expected to change.
- **Action Items**: open commitments until done or cancelled.
- **Log**: dated, append-only record of meaningful changes.

### Topic Slug

Topic slugs MUST be lowercase kebab-case using `[a-z0-9-]+`.

Examples:

- `car-maintenance`
- `personal-finance`
- `mom-health`

### Topic Links

Cross-topic links MUST use wiki-link format:

- `[[topic-slug]]`

Only canonical slugs are valid link targets.

## Required Files (under `vault/`)

### `vault/topic-index.md`

The topic index is the authoritative list of topics for routing.

Each entry MUST use:

```text
- [[topic-slug]] — <short description>
```

### `vault/templates/topic-template.md`

The template MUST include:

```md
# <Topic Title>

## Stable facts

## Current State

## Action Items

## Log
```

## Source Records

### Source Directory

All source records MUST be stored under one unified tree inside the vault content root:

- `vault/sources/YYYY/YYYY-MM-DD-<source-id>.md`

`<source-id>` is unique per date.

Examples:

- `vault/sources/2026/2026-04-15-001.md`
- `vault/sources/2026/2026-04-15-002.md`

### Source File Content

Each source record MUST include origin metadata in-file.

Minimum required content:

```md
# Source Record: YYYY-MM-DD-<source-id>

- source: <origin description>
- timestamp: <ISO-8601 timestamp>
- message: "<user message content>"
- notes: <optional>
```

Examples of `source` values:

- `Telegram from user xyz`
- `Cursor chat from user xyz`
- `Manual note from user xyz`

## Log Format and Provenance

Log entries MUST be append-only.

Every log entry MUST include a link to the source record file that triggered the change.

Canonical log line format (paths relative to the topic file under `vault/topics/`):

```text
- YYYY-MM-DD — <event summary>. Source: [<source-id>](../sources/YYYY/YYYY-MM-DD-<source-id>.md)
```

Example:

```text
- 2026-04-15 — Updated haircut cadence and overdue status. Source: [2026-04-15-001](../sources/2026/2026-04-15-001.md)
```

## Message Types

Every incoming user message MUST be classified into exactly one top-level type:

1. `kb_interacting` (message participates in knowledge-base flow)
2. `non_kb_interacting` (message does not participate in knowledge-base flow)

Within `kb_interacting`, intent MUST be one of:

1. `query`
2. `update`
3. `query_and_update`

## Processing Rules by Intent

### Query

- Read relevant topic pages under `vault/topics/`.
- Return answer.
- Do not modify topic files.
- Do not append log entries.
- Do not create source records.

### Update

- Route to existing topic(s) or create topic if needed.
- Apply updates to the correct section(s).
- Create a source record under `vault/sources/`.
- Append log entry for each meaningful change, with source link.

### Query and Update

- Perform update behavior first.
- Then answer the query from updated state.
- Log only the change portion; do not log pure question text.

### Non-KB Interacting

- Do not invoke knowledge-base update/query flow.
- Handle message directly according to normal assistant behavior.

## Topic Management Behavior

This behavior is centralized in one skill: `kb-topic-management`.

It MUST cover:

- Topic routing
- Topic creation
- Topic updates
- Topic split handling
- Action item lifecycle updates
- Source record creation for updates
- Log appends with provenance links

### Routing Priority

For KB-interacting messages:

1. Prefer updating an existing topic.
2. Create a new topic only when no suitable topic exists.
3. Ask one clarification question if routing is ambiguous.

### Topic Creation

When creating a topic:

1. Generate canonical `topic-slug`.
2. Create `vault/topics/<topic-slug>.md` using `vault/templates/topic-template.md`.
3. Add entry to `vault/topic-index.md`.
4. Write initial content to correct section(s).
5. Append log entry with source link for substantive updates.

### Topic Update Rules

Classification for placement:

- Stable truth -> `## Stable facts`
- Volatile now-state -> `## Current State`
- Commitment/task -> `## Action Items`
- Meaningful dated change -> append in `## Log`

### Action Item Lifecycle

Action items remain visible until done or cancelled according to project policy.

Completion or cancellation SHOULD generate a log entry with source link when it represents a meaningful state change.

### Topic Split

When one topic contains separable subjects:

1. Define what content moves and what remains.
2. Ask user confirmation if split boundaries are unclear.
3. Create new topic file(s) as needed under `vault/topics/`.
4. Move content to correct destinations.
5. Update `vault/topic-index.md` and cross-links.
6. Append log entry with source links in affected topics for substantive changes.

## Response Delivery Behavior

Response generation and delivery is centralized in one skill: `kb-response`.

This skill MUST run for KB-interacting messages and MUST produce a short, easy-to-read user message.

### Delivery Channel Rules

- If source is Telegram: send final user response through the Telegram command interface (command implemented separately).
- If source is non-Telegram chat: return final response in normal chat.

### Response Content Rules

KB response MUST include only concise essentials:

- Query answer (if any)
- What changed (if any)
- Which topic(s) were touched
- A single clarification question only when needed

## Skill File Specifications

Skills live at the repository root under `.cursor/skills/` (framework layer).

### `.cursor/skills/kb-topic-management/SKILL.md`

Must include:

1. **When to invoke**
   - Any `kb_interacting` message with `update` or `query_and_update`.
2. **Inputs**
   - User message, message source metadata, `vault/topic-index.md`, candidate topics, source id context.
3. **Procedure**
   - classify intent details
   - route/create/update/split under `vault/`
   - update action items
   - create source file under `vault/sources/` for updates
   - append logs with source links
4. **Output to caller**
   - concise machine-usable summary of touched topics and changes (format is implementation-defined).

### `.cursor/skills/kb-response/SKILL.md`

Must include:

1. **When to invoke**
   - Any `kb_interacting` message after KB handling.
2. **Inputs**
   - message source metadata, answer text, update summary, touched topics.
3. **Procedure**
   - compose short user-facing message
   - choose delivery channel (Telegram command or normal chat)
   - deliver response
4. **Constraints**
   - no verbose output
   - one clarifying question at most, only if needed

## AGENTS.md Requirements

`AGENTS.md` at the repository root is the **only** place framework behavioral rules live (no separate `.mdc` or `.cursor/rules/` files for this system). It MUST describe behavior in terms of the `vault/` content paths in this specification and MUST include the following.

### Message classification

- Top-level classification: `kb_interacting` vs `non_kb_interacting`.
- KB intent classification: `query`, `update`, `query_and_update`.
- Requirement that exactly one intent path is selected per message.

### Topic management

- Required topic heading order (`## Stable facts` through `## Log`).
- Placement logic by knowledge type (stable facts, current state, action items, log).
- Routing priority and ambiguity handling (aligned with **Topic Management Behavior** and **Processing Rules by Intent** in this document).
- Create, update, split, and action-item lifecycle expectations.
- Statement that durable topic files and the topic index live under `vault/`.

### Provenance and logging

- Unified source tree under `vault/sources/YYYY/`.
- Required source file metadata fields (minimum fields as in **Source File Content**).
- Append-only log behavior and mandatory source links for log entries.
- Rule that pure queries do not create logs or source records.

### Response delivery

- KB responses are short and easy to read.
- Telegram-origin messages use the Telegram send command (command implemented separately).
- Non-Telegram KB messages use normal chat response.
- Response includes answer, changes, and touched topics as applicable.

### Policy summary (cross-cutting)

- Objective of maintaining durable life-admin knowledge under `vault/topics/` (and related `vault/` paths).
- Routing priority: prefer updating existing topics before creating new topics.
- Requirement to invoke `kb-response` for KB-interacting messages after KB handling.
- Clarification that **`AGENTS.md`** and skills apply from the repo root while topic and source files live under `vault/`.

### Git (Cursor Cloud Agent)

- When runs are started without automatic pull request creation at ingress, **`AGENTS.md`** states that substantive edits under `vault/` are delivered as commits **pushed to `main`** (and that pull requests are optional, user-directed only).

## Minimal Acceptance Criteria

Implementation is conformant when all of the following are true:

1. The framework layout includes `.cursor/skills/` and root `AGENTS.md` (containing all rules per **AGENTS.md Requirements**); the knowledge layout exists under `vault/` with `vault/topic-index.md`, `vault/topics/`, `vault/sources/`, and `vault/templates/` as specified.
2. Every topic file under `vault/topics/` contains required headings in required order.
3. Topic links use `[[topic-slug]]` format with canonical slugs.
4. All substantive updates create source record files under `vault/sources/YYYY/`.
5. All new log entries are append-only and include source links.
6. Query-only KB messages never create logs or source files.
7. KB-interacting responses are channel-aware and concise.
8. Topic management behavior is centralized in `kb-topic-management`.
