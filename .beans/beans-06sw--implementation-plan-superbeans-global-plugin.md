---
# beans-06sw
title: 'Implementation Plan: Superbeans Global Plugin'
status: completed
type: task
priority: normal
created_at: 2026-01-07T20:26:16Z
updated_at: 2026-01-07T21:08:35Z
parent: beans-2wjr
---

# Implementation Plan: Superbeans Global Plugin

## Overview

Configure the superbeans repository as a proper Claude Code marketplace plugin, then update dotfiles to load it globally, removing the old scattered hooks/commands.

## Architecture

```
/home/stefan/projects/superbeans/          <- Marketplace root
├── .claude-plugin/
│   ├── plugin.json                        <- Plugin metadata (exists)
│   └── marketplace.json                   <- Points to root plugin (UPDATE)
├── commands/                              <- Slash commands (exists)
├── skills/                                <- Skills (exists)
├── hooks/
│   ├── hooks.json                         <- Add beans prime hooks (UPDATE)
│   └── superbeans-track-state.sh          <- Tmux state tracker (exists)
└── extras/                                <- DELETE (outdated)

~/dotfiles/dotfiles/claude.json            <- Add marketplace + enable plugin
~/dotfiles/dotfiles/claude/
├── commands/bean-*.md                     <- DELETE (moved to plugin)
├── docs/SUPERBEANS.md                     <- DELETE (moved to plugin)
└── hooks/superbeans-track-state.sh        <- DELETE (moved to plugin)
```

## Implementation Phases

| Phase | Bean | Description |
|-------|------|-------------|
| 1 | beans-5w3x | Superbeans repo cleanup (delete extras, fix marketplace.json) |
| 2 | beans-0eti | Add beans prime hooks to plugin |
| 3 | beans-6iwi | Configure dotfiles (add marketplace, enable plugin, remove hooks) |
| 4 | beans-19cx | Delete old dotfiles cruft (commands, docs, hooks) |

## Execution Order

Phases 1-2 modify the superbeans repo (commit together or separately).
Phases 3-4 modify the dotfiles repo (commit together or separately).

Both repos need changes, so work in superbeans first, then dotfiles.