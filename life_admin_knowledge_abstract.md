# Life admin knowledge — abstract model

This document describes a **conceptual** knowledge and administration layer: what to track, how truth changes over time, and how it relates to the Vault layout in [vault_storage.md](vault_storage.md). It does not prescribe a single tool; implementations may be Obsidian-based wikis, agent skills, or other markdown-first workflows.

---

## Purpose

Support **personal life administration** alongside durable work topics:

- Capture **small facts and intentions** quickly (inbox-style input).
- Answer **“what is the status on X?”** and **“what should I do next?”** from stored state, not only from chat memory.
- Let knowledge **evolve**: current state updates, tasks complete, and (optionally) history remains recoverable.

This extends the Vault’s core pattern (durable topics, reference knowledge, records) with explicit rules for **kinds of truth** and **how long each kind is expected to stay true**.

---

## Alignment with the Vault tree

| Abstract concept | Typical Vault placement |
| ---------------- | ----------------------- |
| Stable personal baseline | `me.md`, or dedicated `knowledge/` articles for domains that rarely change |
| Active life areas and ongoing threads | `topics/` using the topic template, or a parallel convention such as `topics/life-*` |
| How the system works (this model, conventions) | `knowledge/*.md` |
| Provenance for changes driven by chat | `records/` |

Life-admin content can live in the same vault as work topics; naming and linking keep domains separable.

---

## Four kinds of truth (lifecycle)

Statements differ by **how long they stay true** and **what happens when they change**. A practical split:

1. **Stable facts** — Slow-changing truths for *this* topic, edited only when reality changes (correction or material update). Not limited to “who someone is”: same lifecycle applies to a lease end date, a car’s model year, an account number, a vendor’s legal name, or a birthday. Rare edits; no implied expiry like “current state.”
2. **Current state** — True *now*, expected to change (e.g. “haircut overdue,” “project phase: planning”). Overwritten when reality moves; not treated as permanent fact.
3. **Commitments / tasks** — Open until done or cancelled. Have completion conditions; once closed, they leave the open list (and may optionally generate a one-line event).
4. **Events / log** — Dated, append-only: what happened, when. Used when **the sequence of changes** matters, not only the latest value.

The same **subject** (e.g. health, finances, a relationship) holds all four on **one durable topic page**—see [Single topic page](#single-topic-page) and [Topic lifecycle](#topic-lifecycle).

**Requirement.** Every durable topic governed by this abstract **includes a place for all four kinds**: labeled sections for stable facts, current state, open actions, and log. A section may be **empty** (use a visible placeholder such as `— none —` or `*Nothing here yet.*`) when nothing applies yet—structure still matters so agents and humans know where new information belongs.

---

## Topic lifecycle

How topics are created, updated, and split. Aligned with the Vault’s `topic-manage` behavior in [vault_runtime.md](vault_runtime.md); details in [vault_storage.md](vault_storage.md) (templates, `index.md`). The filesystem layout does not move topics aside for obsolescence or importance; that tiering is reserved for a separate memory or retention layer if you add one.

### Create

When a **new concern** needs durable tracking (a separable subject in life admin or work), add `topics/<basename>.md`, ensure the basename is unique across `topics/`, `knowledge/`, and `records/`, update `index.md`, and link from related topics or `records/` as appropriate. New topics include the [four section](#single-topic-page) structure from the start (empty placeholders allowed).

### Update

Edit topics **in place**: refresh stable facts, current state, open actions, and log; add decisions, open questions, key people, and references when relevant. Prefer linking to `records/` for provenance when a **chat message** drove the change.

### Split (advisory)

After **substantive** edits, evaluate whether one topic file mixes **two or more separable** subjects (e.g. finances and health merged into one file).

- If **yes**: describe what would become its own topic and what would stay; confirm with the user if choice is required. If approved: create the new file, move content, update `index.md`, fix `[[links]]` elsewhere.
- If **no** or the user declines: leave as-is.

Do not re-prompt split on every tiny edit—only when the topic meaningfully changed or the user asks.

---

## Raw input as sufficient explanation

For this model, **the triggering capture is enough explanation** for a compiled or merged update: the wiki does not require a separate “why” narrative for routine updates.

- Example: the statement *“I need a haircut”* is in scope as **state and/or task**; motivation, psychology, or deep causal analysis are **out of scope** unless the user chooses to add them.
- Provenance is satisfied by **linking or retaining the raw note** (and, if used, version history) that led to the change—not by interrogating intent.

---

## Single topic page

For a coherent subject (person, area of life, project, asset, account), **one durable topic file** uses **all four kinds of truth** as **explicit sections** (empty placeholders allowed—see [Four kinds of truth](#four-kinds-of-truth-lifecycle)):

| Kind | Section heading (recommended) | Role |
| ---- | ----------------------------- | ---- |
| Stable facts | `## Stable facts` | Slow-changing reference truths for this topic. |
| Current state | `## Current State` | Short, overwrite-friendly snapshot of what is true *now* and expected to change (volatile slice; stable material lives under Stable facts). |
| Open actions | `## Action Items` | Checklists or tasks with optional due/urgency; open until done or cancelled. |
| Events / log | `## Log` | Dated, append-only lines when something meaningful changed (see [History](#history)). |

**Vault template alignment.** The baseline topic shape in [vault_storage.md](vault_storage.md#topic-file-structure) already includes `## Current State`, `## Action Items`, and work-oriented sections (`## Decisions`, `## Open Questions`, `## Key People`, `## References`). Topics using this abstract **add** `## Stable facts` (before `## Current State` is a natural order) and `## Log`. **`## Current State`** holds only the **volatile** “now” content for this model; slow-changing truths live under **Stable facts**. Keep Decisions, Open Questions, Key People, and References when they apply; they complement the four kinds, not replace them.

Sections prevent everything from reading as undifferentiated “facts.”

---

## History: when to keep it vs overwrite

- **Overwrite** the volatile content under `## Current State` for readability when only the latest snapshot matters.
- **Append** to a log (or rely on **document versioning** of raw or topic files) when future questions would be: *when did this flip?*, *what did we try before?*, or *how did we get here?*

Trivial churn does not require a log line each time. A hybrid works well: **latest state on the page**, **full history in Git** for files that matter, **short human log** only for non-trivial transitions.

---

## Versioning

Tracking **each raw note or topic file under version control** is compatible with this model: it provides diffs and recovery without requiring a hand-maintained narrative. Semantic “why” for merges or agent steps may still not appear in Git alone; the user’s **raw capture** remains the preferred minimal explanation.

---

## Evolution and status

The system is **not** limited to static reference articles. Topics are expected to:

- Keep **all four section** placeholders or contents current as information arrives.
- Update **current state** and **action items** as life moves; adjust **stable facts** when slow-changing truth changes; append to **log** when history matters.
- Add **decisions** and **references** when something substantive changes.
- Use `records/` when a **chat message** drives a durable edit, preserving the Vault’s provenance rule.

Optional periodic prompts (outside this spec) can reinforce consolidation: e.g. process inbox, refresh status sections, surface next actions—aligned with `review`-style workflows in [vault_runtime.md](vault_runtime.md).

---

## What this abstract does not define

- A specific application, plugin, or LLM product.
- Required automation (scheduling, notifications, mobile clients).
- Psychological or coaching models beyond **storing what the user asserted**.

Implementations choose tooling; this document fixes **categories of knowledge**, **per-topic structure (four kinds + lifecycle)**, **scope of explanation**, and **fit with the Vault filesystem**.

---

## See also

- [vault.md](vault.md) — hub and principles.
- [vault_storage.md](vault_storage.md) — directories, topic template, records.
- [vault_runtime.md](vault_runtime.md) — skills, mutating vs read-only, git, `topic-manage` (split advisory).
