# Life admin knowledge — abstract model

## Overview

This abstract describes a personal knowledge representation and management system to organize, capture, and maintain durable personal and work knowledge in a structured, actionable way.
Knowledge is represented as a set of topic files, each organized by the four kinds of truth: stable facts, current state, action items, and an append-only event log.
This representation makes the information easier to find, update, and act upon by clearly separating long-term facts, the current situation, actionable items, and historical events within each topic.

---

## Goals

Support **personal life administration** alongside durable work topics:

- Capture **facts and intentions** quickly.
- Answer **“what is the status on X?”** and **“what should I do next?”** from stored state, not only from chat memory.
- Let knowledge **evolve**: current state updates, action items complete, and (optionally) history remains recoverable.
- The original source data is preserved for provenance whenever new information is received and integrated, so no information is lost.

---

## Four kinds of truth (lifecycle)

Statements differ by **how long they stay true** and **what happens when they change**. A practical split:

1. **Stable facts** — Slow-changing truths for *this* topic, edited only when reality changes (correction or material update). Not limited to “who someone is”: same lifecycle applies to a lease end date, a car’s model year, an account number, a vendor’s legal name, or a birthday. Rare edits; no implied expiry like “current state.”
2. **Current state** — True *now*, expected to change (e.g. “haircut overdue,” “project phase: planning”). Overwritten when reality moves; not treated as permanent fact.
3. **Action items** — Open until done or cancelled. Have completion conditions; once closed, they leave the open list (and may optionally generate a one-line event).
4. **Events / log** — Dated, append-only: what happened, when. Used when **the sequence of changes** matters, not only the latest value.

The same **subject** (e.g. health, finances, a relationship) holds all four on **one durable topic page**—see [Single topic page](#single-topic-page) and [Topic lifecycle](#topic-lifecycle).

**Requirement.** Every durable topic governed by this abstract **includes a place for all four kinds**: labeled sections for stable facts, current state, action items, and log.

---

## Topic lifecycle

How topics are created, updated, and split. The file layout can stay flat by default; obsolescence and importance tiering can be handled in a separate memory or retention layer if needed.

### Create

When a **new concern** needs durable tracking (a separable subject in life admin or work), create a new topic file with a unique basename, update any topic index you maintain, and add links from related topics or source records as appropriate. New topics include the [four-section](#single-topic-page) structure from the start.

### Update

Edit topics **in place**: refresh stable facts, current state, action items, and log; add decisions, open questions, key people, and references when relevant. Prefer linking to source records for provenance when a chat message or event drove the change.

### Split (advisory)

After **substantive** edits, evaluate whether one topic file mixes **two or more separable** subjects (e.g. finances and health merged into one file).

- If **yes**: describe what would become its own topic and what would stay; confirm with the user if choice is required. If approved: create the new file, move content, update your topic index, and fix `[[links]]` elsewhere.
- If **no** or the user declines: leave as-is.

Do not re-prompt split on every tiny edit—only when the topic meaningfully changed or the user asks.

---

## Knowledge as Wiki pages

A **wiki-style page** (one file per coherent subject, with easily browsable and linkable sections) is essential for robust knowledge management:

- **Easy information routing:** Each topic is a clear destination for related facts, updates, and questions. When new information arrives, it’s obvious where it should be recorded—reducing ambiguity and minimizing lost context.
- **Improved discoverability:** Flat, clearly named topic files and consistent section headings make it simple to find what you’re looking for—either by filename or by section. References (`[[wiki-links]]`) and a unified topic index strengthen cross-topic connections and visibility.
- **Supports incremental improvement:** Topics can be started with a seed fact and grow as more details emerge, without requiring up-front structure.
- **Enhanced linking and navigation:** Wiki links (e.g. `[[project-x]]`, `[[profile]]`) let humans and agents connect related pages instantly, surfacing dependencies, relationships, and relevant context across the knowledge base.
- **Flexible context windows for agents:** Because every subject has its own durable home, chat-based agents can decide which topics to read, surface, or modify—enabling smarter, more focused interactions.

## Single topic page

For a coherent subject (person, area of life, project, asset, account), **one durable topic file** uses **all four kinds of truth** as explicit sections:


| Kind          | Section heading (recommended) | Role                                                                                                                                        |
| ------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Stable facts  | `## Stable facts`             | Slow-changing reference truths for this topic.                                                                                              |
| Current state | `## Current State`            | Short, overwrite-friendly snapshot of what is true *now* and expected to change (volatile slice; stable material lives under Stable facts). |
| Action items  | `## Action Items`             | Open commitments with optional due/urgency; remain open until done or cancelled.                                                             |
| Events / log  | `## Log`                      | Dated, append-only lines when something meaningful changed (see [History and updates](#history-and-updates-overwrite-vs-log)).              |


## History and updates: overwrite vs log

Each topic file captures both current truth and meaningful change over time. Use the sections as follows:

- Use the `## Current State` section for whatever is true *right now*—update or overwrite this as things change. Only the latest information matters here; it’s not meant to track history.
- Use the `## Log` section to append short, dated entries when it’s important to remember **when** something changed or what sequence of events led to the current state. For example, log: “2023-05-01 — Switched email provider,” or “2024-03-17 — Alarm system installed.”
- Use document versioning (if available) to preserve fine-grained edit history without copying every minor update into the log.

**Operating approach:** Keep `## Current State` and `## Action Items` current, update `## Stable facts` when slow-changing truth changes, and append to `## Log` when the sequence or timing of changes matters.

Topics are expected to:

- Keep all four sections current as information arrives.
- Update **current state** and **action items** as life moves; adjust **stable facts** when slow-changing truth changes; append to **log** when history matters.
- Add **decisions** and maintain **references**—including links to source data or relevant context—whenever something substantive changes.
