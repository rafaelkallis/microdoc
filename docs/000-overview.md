---
description: Project overview. Claude Code plugin that injects doc summaries into sessions via SessionStart hook. Stdlib only, configurable globs.
---

# microdoc

A Claude Code plugin that makes project documentation accessible in every session. It scans markdown files, extracts short descriptions from YAML frontmatter, and injects them as structured XML context at session start.

## Purpose

Claude Code doesn't automatically know about your project's docs. microdoc bridges that gap with a lightweight RAG-like approach: frontmatter descriptions act as an index, and Claude reads full docs on demand when relevant.

## Key Components

- **SessionStart hook** (`plugins/microdoc/hooks/microdoc.js`): Glob-matches doc files, parses frontmatter, outputs XML to stdout. Uses Node.js stdlib only -- no dependencies. Two file discovery strategies: git-aware (primary, via `git ls-files`) and filesystem fallback for non-git projects.
- **microdoc-development skill**: Guides writing token-efficient descriptions (15-20 word topic indexes).
- **microdoc-init skill**: Bootstraps microdoc in a project -- creates docs directory, seeds overview, backfills missing descriptions. Invoke with `/microdoc-init`.

## Configuration

Two environment variables control behavior:

- `CLAUDE_MICRODOC_GLOB`: Comma-separated glob patterns for doc files (default: `docs/**/*.{md,mdc}`).
- `CLAUDE_MICRODOC_DISABLED`: Set to `1` to disable the plugin.

## Distribution

Published as a Claude Code plugin marketplace. Install with:

```sh
/plugin marketplace add rafaelkallis/microdoc
/plugin install microdoc@microdoc
```
