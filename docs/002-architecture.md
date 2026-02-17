---
description: Architecture and file layout. Hook pipeline, git-aware and fallback file discovery, frontmatter parser, glob engine, XML output, manifests, skills.
---

# Architecture

## Directory Layout

```
plugins/microdoc/
  .claude-plugin/plugin.json    Plugin manifest (name, version, description)
  hooks/
    hooks.json                  Registers microdoc.js as a SessionStart hook
    microdoc.js                 Main hook script
  skills/
    microdoc-author/            Skill for writing/maintaining doc descriptions
    microdoc-audit/             Skill for bulk review of doc descriptions
  test/
    unit.test.js                Unit tests for exported helpers
    integration.test.js         Integration tests spawning the hook script
.claude-plugin/
  marketplace.json              Marketplace manifest (plugin registry)
docs/                           Project documentation (consumed by microdoc itself)
```

## Hook Script Pipeline

`plugins/microdoc/hooks/microdoc.js` runs as a SessionStart hook. The pipeline:

1. **Early exit** if `CLAUDE_PROJECT_DIR` is unset or `CLAUDE_MICRODOC_DISABLED=1`.
2. **Parse glob patterns** from `CLAUDE_MICRODOC_GLOB` (default: `docs/**/*.{md,mdc}`). `splitGlobs` splits on commas while respecting brace nesting. Each pattern is compiled to a regex via `globToRegex`.
3. **Discover files** using one of two strategies (see below).
4. **Filter** discovered files against the compiled regexes, deduplicate, sort.
5. **Extract descriptions** from YAML frontmatter of each matching file.
6. **Output XML** to stdout -- Claude Code captures this and injects it into the session context.

## File Discovery

Two strategies, selected automatically:

### Git-aware (primary)

Runs `git ls-files --cached --others --exclude-standard` in `CLAUDE_PROJECT_DIR`. This returns tracked files plus untracked-but-not-ignored files, respecting `.gitignore`. Fast and accurate for git repos.

### Filesystem fallback

When `git ls-files` fails (not a git repo, git not installed), falls back to `readdirRecursive`. This walks the directory tree starting from each glob's static prefix (extracted by `extractStaticPrefix`). Skips hardcoded directories: `.git`, `node_modules`, `.next`, `.nuxt`, `dist`, `build`, `.turbo`, `.cache`.

## Glob Engine

Custom minimal glob-to-regex compiler (`globToRegex`). Supports:

- `**` -- matches any path segments (zero or more directories)
- `*` -- matches anything except `/`
- `?` -- matches a single non-`/` character
- `{a,b}` -- alternation (brace expansion)
- Regex special characters in paths are escaped
- Unclosed braces are treated as literal `{`

`extractStaticPrefix` pulls the literal path prefix before any wildcard character (`*`, `?`, `{`, `[`), used to narrow the filesystem scan root.

## Frontmatter Parser

`extractDescription` is a minimal parser, not a full YAML implementation. Rules:

- Frontmatter must start at byte 0 with `---\n` and end with `\n---`.
- Scans for a `description:` key on its own line.
- Supports inline values (`description: some text`), quoted strings (single/double), and block scalars (`|`, `>` with optional chomp indicators).
- Returns `null` if no valid description is found.

## XML Output Format

```xml
<microdoc>
<attribution>...</attribution>
<instructions>...</instructions>

<docs>
<doc>
<path>docs/example.md</path>
<description>Description text here</description>
</doc>
</docs>
</microdoc>
```

Descriptions are XML-escaped (`&`, `<`, `>`) via `xmlEscape`. Files without a description show `(no description)`.

## Development Constraints

- **stdlib only** -- `fs`, `path`, `child_process`. No npm dependencies.
- Hook has a 5-second timeout (configured in `hooks.json`).
- All helper functions are exported from `microdoc.js` for unit testability.
- A `require.main === module` guard enables both direct CLI execution and `require()` for tests.
