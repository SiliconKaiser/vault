---
name: vault-glossary-detect
description: >-
  Detects domain-specific or org-internal terms not in the glossary during any
  conversation. Asks the user for a definition via AskQuestion, then appends to
  knowledge/glossary.md.
---

# Vault: glossary-detect

Read `knowledge/glossary.md` at the knowledge root.

1. **Detect** -- During parsing or conversation, identify terms (acronyms, code names, jargon) that are not in `knowledge/glossary.md`. Ignore common technical vocabulary the AI already understands (e.g. API, DNS, Redis, Kubernetes).
2. **Ask** -- For each unknown term, use `AskQuestion` with options:
   - One or more candidate definitions if the AI can infer one from context
   - "Skip"
3. **Write** -- Append confirmed terms to `knowledge/glossary.md` in alphabetical order. No second confirmation needed.
