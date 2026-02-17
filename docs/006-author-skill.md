---
description: Author skill design. Proactive guidance for writing and maintaining individual doc descriptions. Owns the canonical style guide.
---

# Author Skill

The `microdoc-author` skill provides proactive guidance whenever a doc in `docs/` is created, updated, or has its description written.

## Purpose

Individual doc descriptions need to follow a consistent style to keep session context concise and useful. The author skill fires automatically when docs are touched -- by the user or by the agent as part of another task -- ensuring descriptions are written correctly from the start.

## When It Triggers

- **Proactively**: Whenever a doc is created, updated, or has its description written, whether initiated by the user or by the agent during another task
- **On explicit request**: "create a doc", "add a new doc", "write a design record", "update a doc", "update doc description", "initialize microdoc", "set up docs", "backfill descriptions"

## Style Guide Ownership

The author skill owns the canonical description style guide: topic-first, 15-20 words, index not summary. Other skills (including audit) delegate to the author skill's rules rather than duplicating them. This keeps description guidance in one place.

## Differentiation from Audit

**Author** handles individual docs as they are written. It fires frequently and proactively, preventing bad descriptions before they enter the codebase. **Audit** (`microdoc-audit`) scans all docs in bulk for staleness, run infrequently on explicit request. They complement each other: author prevents bad descriptions, audit catches ones that drifted over time.
