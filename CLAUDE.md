# microdoc

Claude Code plugin marketplace that distributes the microdoc plugin.

## How it works

A SessionStart hook scans for doc files matching a configurable glob pattern (default: `docs/**/*.{md,mdc}`), extracts YAML frontmatter `description` fields, and injects them as structured XML into the session context. A companion skill guides writing token-efficient descriptions.

## Versioning

Always bump the plugin version in `plugins/microdoc/.claude-plugin/plugin.json` when making changes. Use semver: patch for fixes, minor for features, major for breaking changes.

