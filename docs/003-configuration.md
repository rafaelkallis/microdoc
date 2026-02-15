---
description: Configuration reference. Environment variables for disabling plugin, custom glob patterns, glob syntax, and where to set them.
---

# Configuration

microdoc is configured entirely through environment variables. No config files are needed.

## Environment Variables

### `CLAUDE_MICRODOC_DISABLED`

Set to `1` to disable the plugin for a project. The hook exits silently and injects nothing.

```sh
CLAUDE_MICRODOC_DISABLED=1
```

### `CLAUDE_MICRODOC_GLOB`

Comma-separated glob patterns for doc file discovery. Defaults to `docs/**/*.{md,mdc}`.

```sh
CLAUDE_MICRODOC_GLOB="docs/**/*.{md,mdc}"
```

Multiple patterns:

```sh
CLAUDE_MICRODOC_GLOB="docs/**/*.md,notes/**/*.md,design/**/*.mdc"
```

Supported glob syntax:

| Pattern | Meaning |
|---------|---------|
| `**`    | Any path segments (zero or more directories) |
| `*`     | Anything except `/` |
| `?`     | Single non-`/` character |
| `{a,b}` | Alternation (brace expansion) |

Commas inside braces are not treated as pattern separators.

## Where to Set Variables

### Per-project (recommended)

Add to `.claude/settings.json` in the project root:

```json
{
  "env": {
    "CLAUDE_MICRODOC_GLOB": "docs/**/*.md,design/**/*.md"
  }
}
```

### Shell

Export before launching Claude Code:

```sh
export CLAUDE_MICRODOC_DISABLED=1
```

### Manual hook testing

Pass directly when running the hook script:

```sh
CLAUDE_PROJECT_DIR=. CLAUDE_MICRODOC_GLOB="**/*.md" node plugins/microdoc/hooks/microdoc.js
```

## Implicit Variables

### `CLAUDE_PROJECT_DIR`

Set automatically by Claude Code to the project root directory. The hook reads this to locate doc files. Not user-configurable -- if unset (e.g., during manual testing), the hook exits silently.
