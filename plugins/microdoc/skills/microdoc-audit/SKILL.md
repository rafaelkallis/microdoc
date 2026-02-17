---
name: microdoc-audit
description: This skill should be used when the user asks to "audit docs", "audit descriptions", "review doc descriptions", "check descriptions", "find stale descriptions", "find missing descriptions", or "validate docs".
---

# Doc Audit

Scan all docs for missing or stale descriptions and fix them in bulk. Follow these steps in order.

## Audit vs Author

The **audit** skill is a deliberate bulk review across all docs, run infrequently on explicit request. It scans every doc, classifies description health, and applies fixes after user confirmation. The **author** skill (`microdoc:microdoc-author`) provides proactive guidance for individual docs as they are written or edited.

## Step 1: Resolve glob

Check the `CLAUDE_MICRODOC_GLOB` environment variable. Extract the base directory from the glob pattern (the part before any `*`, `?`, or `{`). If the variable is not set, default to `docs/` as the base directory (from the default glob `docs/**/*.{md,mdc}`).

## Step 2: Bootstrap if needed

Check if the base directory exists at the project root.

- If it does not exist, create it.
- If it exists and is empty (or was just created), seed it with an `overview.md` file:
  - Gather project context by reading whichever of these files exist at the project root (skip any that don't exist): `README.md`, `CLAUDE.md`, `package.json`.
  - Create the file with YAML frontmatter containing a `description` field following the microdoc:microdoc-author skill's style guide (15-20 words, topic-first, index not summary).
  - The body should provide a concise project overview based on the gathered context.

## Step 3: Discover files

Find all files matching the resolved glob pattern(s) in the base directory.

## Step 4: Classify each file

Read each discovered file and classify it into one of these categories:

| Status | Condition |
|---|---|
| `missing-frontmatter` | File has no YAML frontmatter block (no `---` delimiters at file start) |
| `missing-description` | File has frontmatter but no `description` field, or the field is empty |
| `stale-description` | Description exists but is outdated or poorly written (see staleness criteria) |
| `ok` | Description exists and accurately indexes the doc's current content |

### Staleness criteria

A description is stale if any of the following apply:

- It mentions topics or terms no longer present in the doc body
- It omits the doc's primary topic or key terms
- It exceeds 25 words
- It reads as a prose summary rather than a topic index (full sentences, justifications, implementation details)

When in doubt, flag as stale. False positives are cheap (the user reviews before changes are applied).

## Step 5: Report

Print a markdown table with columns: **File**, **Status**, **Issue**.

Follow the table with a one-line summary: how many files total, how many have issues.

If zero files have issues, print "All descriptions are up-to-date." and stop here.

## Step 6: Confirm

Ask the user whether to proceed with fixing the issues. Use AskUserQuestion with options to apply fixes or stop. Do not apply any changes without user consent.

## Step 7: Apply fixes

For each file with an issue:

1. Read the full file content.
2. Draft a description following the microdoc:microdoc-author skill's style guide:
   - Lead with the topic in 2-3 words
   - State the key decision or purpose without justification
   - List key terms that would trigger reading the full doc
   - Target 15-20 words
3. Add or update the YAML frontmatter `description` field. Preserve all other frontmatter fields.
   - If adding frontmatter to a file that has none, insert a `---` / `description: ...` / `---` block at the top of the file.

## Step 8: Summary

Print a list of every file that was updated, with its new description.
