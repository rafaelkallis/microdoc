---
description: Testing strategy and commands. Unit tests for helpers, integration tests for hook with git-aware and fallback discovery. CI matrix.
---

# Testing

## Running Tests

Run the full test suite:

```sh
node --test
```

Run an individual test file:

```sh
node --test plugins/microdoc/test/unit.test.js
```

Prefer relative paths over absolute paths when executing commands with path arguments.

## Manual Hook Testing

Test the hook against a project directory:

```sh
CLAUDE_PROJECT_DIR=<project-with-docs> node plugins/microdoc/hooks/microdoc.js
```

Test with the plugin disabled:

```sh
CLAUDE_MICRODOC_DISABLED=1 CLAUDE_PROJECT_DIR=<project-with-docs> node plugins/microdoc/hooks/microdoc.js
```

## Test Framework

Tests use `node:test` and `node:assert/strict` -- no external dependencies, consistent with the stdlib-only constraint.

## Test Structure

- `plugins/microdoc/test/unit.test.js` -- Unit tests for exported helpers: `xmlEscape`, `globToRegex`, `splitGlobs`, `extractStaticPrefix`, `extractDescription`, `readdirRecursive`.
- `plugins/microdoc/test/integration.test.js` -- Integration tests that spawn the hook script as a child process against temporary directories. Covers three scenarios:
  - **Default (git-based)**: Tests within a `git init`'d temp dir to exercise `git ls-files` file discovery, `.gitignore` respect, and untracked file inclusion.
  - **Fallback (non-git)**: Tests in a plain temp dir to exercise `readdirRecursive` with `SKIP_DIRS` filtering (e.g., `node_modules`).
  - **General**: XML output structure, disabled mode, missing `CLAUDE_PROJECT_DIR`, custom globs, multi-glob patterns, deeply nested files, XML escaping, missing descriptions.

## CI

GitHub Actions workflow at `.github/workflows/test.yml` runs `node --test` across a matrix of Node.js versions: 22, 24, 25.

### Node.js Version Policy

CI tests against 3 Node.js versions: the Current release and the two most recent LTS versions. If the Current release is itself an LTS version, all three are LTS. For example: 22, 24, 25 today; once 26 ships, 22, 24, 26.

## Writing New Tests

- Unit tests go in `unit.test.js`. Import helpers from `../hooks/microdoc.js` and test pure functions directly.
- Integration tests go in `integration.test.js`. Use the `writeDoc` / `runScript` pattern: create files in a temp dir, spawn the hook, assert on stdout.
- Each `describe` block manages its own temp directory via `before`/`after` hooks with `fs.mkdtempSync` and `fs.rmSync`.
