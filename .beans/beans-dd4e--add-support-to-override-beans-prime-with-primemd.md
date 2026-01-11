---
# beans-dd4e
title: Add support to override beans prime with prime.md
status: completed
type: feature
priority: normal
created_at: 2026-01-04T23:04:19Z
updated_at: 2026-01-05T03:12:10Z
---

Allow users to configure how beans/superbeans behaves by providing a custom `prime.md` (or similar) file that overrides or extends the default `beans prime` output.

## Motivation

The default prime prompt works well for most cases, but projects may have specific conventions, workflows, or additional context that should be included when priming AI agents. A custom prime file would allow:

- Project-specific bean conventions (e.g., custom types, status workflows)
- Additional instructions for how to use beans in this particular codebase
- Override default behaviors that don't fit the project's needs

## Design Decisions

- [x] Should the custom file completely replace the default, or merge/extend it?
  - **Always override** - no merge logic
- [x] What should the file be named?
  - **`.beans-prime.md`** in current project/worktree root (next to `.beans.yml`)
  - Read from cwd/project root, independent of where `.beans/` data lives
- [x] Should there be template variables available in the custom file?
  - **No** - skip for v1 (YAGNI)

## Implementation

- [x] Check for `.beans-prime.md` in project root before rendering default template
- [x] If found, output its contents directly instead of the default prompt
- [x] Add tests for the override behavior
- [x] Document in README.md
- [x] Update `beans prime --help` text

## Summary of Changes

- Added `CustomPrimeFilename` constant (`.beans-prime.md`) in `cmd/prime.go`
- Modified `beans prime` to check for `.beans-prime.md` in project root before rendering default template
- If custom file exists, its contents are output directly (full override, no merge)
- Added tests in `cmd/prime_test.go`
- Documented in README.md ("Customizing the Prime Prompt" section)
- Updated `beans prime --help` long description