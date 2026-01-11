---
# beans-erio
title: 'SUPERBEANS: beans prime should mention beans location and be worktree-aware'
status: completed
type: feature
priority: normal
created_at: 2026-01-02T18:57:01Z
updated_at: 2026-01-02T19:02:35Z
---

## Problem

When an agent runs `beans prime`, it gets instructions on how to use beans, but doesn't know *where* the beans are stored. This matters especially in worktree setups where:

1. Multiple worktrees share a single `.beans/` directory (via the `WORKTREE` path config)
2. The agent might be confused about which repo/directory the beans belong to
3. The agent can't easily verify it's looking at the right beans

## Proposed Changes

Add a new section near the top of `beans prime` output:

```
## Beans Location

Beans are stored at: `/home/stefan/coding/beans/.worktrees/beans/.beans/`
Config file: `/home/stefan/coding/beans/.beans.yml`

**Worktree mode**: You are in a git worktree. Beans are shared with the main repository at `/home/stefan/coding/beans`.
```

(The worktree note only appears when in a worktree)

## Implementation

1. Add fields to `promptData` struct in `cmd/prime.go`:
   - `BeansPath string` - resolved absolute path to beans directory
   - `ConfigPath string` - path to `.beans.yml` config file
   - `IsWorktree bool` - whether we're in a worktree
   - `MainRepoRoot string` - main repo root (if in worktree)

2. Populate these in the `RunE` function using:
   - `cfg.ResolveBeansPath()` for the beans path
   - `config.GetMainRepoRoot()` for worktree detection
   - Compare current working directory to main repo to detect worktree status

3. Update `prompt.tmpl` to include the location section

## Tasks

- [x] Add new fields to `promptData` struct
- [x] Load config and populate location fields in `RunE`
- [x] Detect worktree status
- [x] Update `prompt.tmpl` with location section
- [x] Test from main repo and from worktree

## Summary of Changes

- Added `BeansPath`, `ConfigPath`, `IsWorktree`, and `MainRepoRoot` fields to `promptData` struct in `cmd/prime.go`
- Updated `RunE` function to load config and detect worktree status by comparing cwd to main repo root
- Added "Beans Location" section to `prompt.tmpl` that shows:
  - The resolved path where beans are stored
  - The config file path (if available)
  - A "Worktree mode" notice when running from a git worktree (with main repo path)