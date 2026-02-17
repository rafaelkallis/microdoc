# microdoc

Claude Code plugin that injects concise doc summaries into every session.

## Why

Claude Code doesn't automatically know about your project's documentation. You could paste full docs into context, but that wastes tokens. microdoc is a super simple RAG pipeline: your markdown docs are the knowledge base, YAML frontmatter descriptions are the index, and Claude reads the full doc on demand when it's relevant.

At session start, the plugin scans your docs, extracts short descriptions from frontmatter, and injects them as structured context. Claude then knows what documentation exists and can retrieve the full content when needed.

## Quick Start

microdoc works out of the box -- install it, and any markdown file under `docs/` with a `description` in its YAML frontmatter gets picked up automatically.

Add a description to your docs:

```yaml
---
description: Document parsing. Chose Docling for PDF/DOCX. Plain read for txt/md.
---
```

At session start, microdoc injects a summary of all your docs into the context:

```xml
<microdoc source="microdoc plugin by Rafael Kallis">
<instructions>...</instructions>
<docs>
  <doc path="docs/architecture/parsing.md">Document parsing. Chose Docling for PDF/DOCX. Plain read for txt/md.</doc>
  <doc path="docs/deployment/staging.md">Staging environment. Docker Compose on EC2. Auto-deploy from main branch.</doc>
</docs>
</microdoc>
```

Claude sees this index and reads the full doc whenever a task overlaps with a description.

## Skills

- **`/microdoc-author`** -- Guides you through writing and maintaining frontmatter descriptions. Covers style, length, and what makes a description effective. Use it when adding or updating a doc.
- **`/microdoc-audit`** -- Bulk-reviews all docs for missing or stale descriptions. Reports a summary table of issues and applies fixes only after your confirmation. Run it periodically to keep descriptions in shape.

## Installation

Add the marketplace and install the plugin:

```sh
/plugin marketplace add rafaelkallis/microdoc
/plugin install microdoc@microdoc
```

## How It Works

1. A **SessionStart hook** runs `hooks/microdoc.js` when a Claude Code session begins.
2. The script glob-matches doc files (default: `docs/**/*.{md,mdc}`) under your project directory.
3. It extracts the `description` field from each file's YAML frontmatter.
4. It outputs structured XML that gets injected into the session context.

No dependencies -- the hook uses Node.js stdlib only.

## Configuration

Set environment variables in `.claude/settings.json` under the `env` field, or export them in your shell.

| Variable | Default | Description |
|---|---|---|
| `CLAUDE_MICRODOC_GLOB` | `docs/**/*.{md,mdc}` | Comma-separated glob patterns for doc files. Supports `**`, `*`, `?`, `{a,b}`. |
| `CLAUDE_MICRODOC_DISABLED` | (unset) | Set to `1` to disable the plugin for a project. |

Example with a custom glob:

```json
{
  "env": {
    "CLAUDE_MICRODOC_GLOB": "documentation/**/*.md,design-records/**/*.md"
  }
}
```

## License

MIT
