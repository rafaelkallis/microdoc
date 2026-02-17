---
description: Audit skill design. Bulk review of all doc descriptions for staleness. Replaces init and curate skills. Confirmation gate before changes.
---

# Audit Skill

The `microdoc-audit` skill is a single entry point for bulk-reviewing doc descriptions and bootstrapping microdoc in a new project.

## Why It Replaces `microdoc-init` and `microdoc-curate`

The init skill handled two tasks: creating the docs directory with a seed file, and backfilling missing descriptions. Both are subsets of auditing. The curate skill added staleness detection but had overlapping triggers with the development skill. Audit subsumes all of init's and curate's functionality under a clearer name with tighter trigger boundaries.

## Workflow

1. Resolve the docs glob and base directory
2. Bootstrap (create directory, seed overview) if needed
3. Discover all matching files
4. Classify each file: missing frontmatter, missing description, stale description, or ok
5. Report a summary table
6. Ask for user confirmation before applying changes
7. Draft and apply descriptions following the style guide
8. Print a summary of changes

## Staleness Detection

Classification is heuristic, performed by Claude when reading each doc. A description is flagged as stale when it:

- References topics no longer in the doc body
- Omits the doc's primary topic
- Exceeds 25 words
- Reads as a prose summary rather than a topic index

The bias is toward flagging. False positives are cheap because the user reviews the report before any changes are applied. False negatives (missing a stale description) are more costly because they leave bad descriptions in place, reducing the quality of every session's context injection.

## Confirmation Gate

The skill reports issues first and only applies fixes after the user consents. This prevents unwanted edits and gives the user a chance to exclude specific files or override the classification.

## Style Guide Delegation

The skill references the `microdoc-author` skill's style guide rather than duplicating the rules. This keeps description guidance in one place and ensures audit and author always agree on what a good description looks like.

## Differentiation from Author

**Audit** scans all docs in bulk for staleness, run infrequently on explicit request. **Author** (`microdoc-author`) handles individual docs as they are written, firing proactively and frequently. They complement each other: author prevents bad descriptions, audit catches ones that drifted over time.
