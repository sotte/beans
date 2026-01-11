---
# beans-risy
title: Integrate superbeans TUI into beans CLI
status: completed
type: feature
priority: normal
created_at: 2026-01-05T20:52:33Z
updated_at: 2026-01-05T21:23:12Z
---

Add a `beans super` command that launches the superbeans TUI directly from the beans CLI.

## Motivation

Currently two separate binaries are needed:
- `beans` (~28MB)
- `superbeans` (~28MB)

Having a single binary would save ~28MB of disk space and simplify distribution/installation.

## Proposed Solution

Add a `super` subcommand to the beans CLI that launches the TUI.

## Summary of Changes

- Created `cmd/super.go` - new command that adds `beans super` (with `superbeans` alias)
- Moved `superbeans/internal/tui/` to `internal/supertui/` to resolve Go's internal package import restriction
- Updated `superbeans/cmd/root.go` to use new import path
- Removed empty `superbeans/internal/` directory