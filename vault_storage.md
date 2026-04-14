# Vault: Storage

Filesystem layout, lifecycle rules, root reference files, and file templates. Hub: [vault.md](vault.md). Behavior and skills: [vault_runtime.md](vault_runtime.md).

The tree below is **knowledge content**. Its root can be a different path or repository than the framework docs; `/vault` here means that root folder (name it however you like). Framework vs content: [vault.md](vault.md#framework-vs-knowledge-content).

---

## Directory layout

```
/vault
  /topics/               -- durable topics: active work (source of truth); see "Durable topics vs reference knowledge"
  /knowledge/
    *.md                 -- reference knowledge (stable articles; how systems work, how-tos, configuration)
  /records/              -- one file per user message that changed vault knowledge state (provenance)
  me.md                  -- personal context: role, team, company (stable reference)
  index.md               -- topic manifest, recent activity log
```

Three top-level directories (`topics`, `knowledge`, `records`). Two root files (`me.md`, `index.md`).

**Retention.** Durable topics stay under `topics/` for the life of the vault unless you split, merge, or delete them by editorial choice. This layout does **not** define moving topics aside to mark them obsolete or less important; that kind of tiering belongs in a separate memory or retention layer if you add one later.

### Durable topics vs reference knowledge

Two different kinds of Markdown, in two different places:

- **`/topics/`** holds **durable topics** (active work): projects, initiatives, tracked concerns. These use the topic file template (Current State, Decisions, Open Questions, Action Items, Key People, References). They change often and carry current state, decisions, active work, and references over time.
- **`/knowledge/*.md`** holds **reference knowledge**: how systems work, configuration, how-tos, comparisons. One file per subject, no required template, no durable-topic lifecycle. A durable topic may link to these files, but there is no 1:1 mapping. Do not call these "topics" in conversation when you mean durable topics; call them **reference knowledge files** (or **knowledge articles**) to avoid confusion with `/topics/`.

### Records (provenance for mutating messages)

**Path:** `/records/*.md`

**When:** Create a new record file only when a user message leads to a **change in stored vault knowledge** (new or edited topics, reference knowledge, or `me.md` / `index.md` updates that reflect durable context). **Do not** create a record for read-only queries (questions answered without editing files).

**Why:** The user interface is often chat-only. Records tie each durable change back to the exact user message that prompted it and list what changed.

**Filename:** `YYYY-MM-DD-<slug>.md` where `<slug>` is a short kebab-case hint (e.g. `2026-04-12-redis-cache-decision.md`). If the basename would collide, append a disambiguator (e.g. `-2`).

Each record uses the template in [Record file structure](#record-file-structure). Topic `References` sections may link to records via `[[basename]]` the same way as other vault files.

### What lives where


| Artifact               | Location                    | Lifecycle                                                    |
| ---------------------- | --------------------------- | ------------------------------------------------------------ |
| Change provenance      | `/records/`                 | One new file per mutating user message; immutable once written |
| Durable knowledge      | `/topics/`                  | Accumulates over time, edited in place                       |
| Reference knowledge    | `/knowledge/*.md`         | Stable domain reference, edited when facts change            |
| Open items / questions | Inside the relevant topic   | Removed when resolved                                        |
| Action items           | Inside the relevant topic   | Updated in place; checked off or removed when done           |
| Message suggestions    | Conversation / chat reply   | Ephemeral guidance during a session                          |
| Personal context       | `me.md`                     | Updated when role, team, or company context changes          |


### Reference files

On startup, the AI reads `me.md` and `index.md` to orient. It reads files under `knowledge/` as needed during processing.

**`me.md`** -- personal context: role, team, company, expertise, and communication preferences. Freeform sections, updated infrequently. The AI uses this to calibrate tone, understand org relationships, and avoid asking obvious questions.

**Other `knowledge/*.md` files** -- reference knowledge, one subject per file (for example how a system works, bucket config, or local dev setup). Not durable topics: no required topic template and no durable-topic lifecycle. Durable topics under `/topics/` may link to them; keep basenames distinct from durable topic and record basenames so `[[wiki-style]]` links resolve unambiguously (see [vault_runtime.md](vault_runtime.md#linking)).

### Topic file structure

```markdown
# <Topic Name>

## Current State
What is true right now.

## Decisions
- YYYY-MM-DD: What was decided (source: [[record-or-topic]])

## Open Questions
- Unresolved items

## Action Items
- [ ] Task (owner, urgency)

## Key People
- @person: role/relevance

## References
- [[YYYY-MM-DD-short-slug]]
```

Message suggestions, when useful, happen in the chat reply. Topic files focus on durable state, decisions, open questions, action items, key people, and references.

### Record file structure

```markdown
# <Short title>

**Date**: YYYY-MM-DD

## User message

<Verbatim or faithfully quoted user message that prompted the change.>

## Vault changes

- paths and one-line description per changed or created file

## Summary

What changed in knowledge state (durable topics, reference articles, or root reference files).

## Related topics

- [[topic-name]]
```

### index.md

```markdown
# Vault

## Topics
- [[topic-name]] - one-line description

## Recent activity
- YYYY-MM-DD: brief note -> topics or files touched (optional pointer to [[record-slug]])
```

On orient, the AI reads `me.md` and `index.md`. The AI maintains `index.md` after each run that changes the vault. It updates `me.md` when it learns new stable personal or org context worth persisting. It adds or updates reference knowledge files under `knowledge/` when long-form reference material should live outside a topic.
