---
# beans-6iwi
title: 'Impl: Configure dotfiles for superbeans marketplace'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-07T20:27:02Z
updated_at: 2026-01-07T20:30:26Z
parent: beans-2wjr
---

# Phase 3: Configure dotfiles for superbeans marketplace

## Files to Modify

1. **Update**: `~/dotfiles/dotfiles/claude.json`

## Steps

### Step 1: Add marketplace configuration

Add `extraKnownMarketplaces` to settings:
```json
"extraKnownMarketplaces": {
  "superbeans-local": {
    "source": {
      "source": "directory",
      "path": "/home/stefan/projects/superbeans"
    }
  }
}
```

### Step 2: Enable superbeans plugin

Add to `enabledPlugins`:
```json
"superbeans@superbeans-local": true
```

### Step 3: Remove superbeans-related hooks

Remove from hooks (now provided by plugin):
- **SessionStart**: Remove `"command": "beans prime"`
- **PreCompact**: Remove `"command": "beans prime"` 
- **PreToolUse**: Remove `"command": "~/.claude/hooks/superbeans-track-state.sh"`
- **Stop**: Remove `"command": "~/.claude/hooks/superbeans-track-state.sh"`

## Commit Message (in dotfiles repo)
```
feat: configure superbeans as marketplace plugin

- Add superbeans-local marketplace pointing to ~/projects/superbeans
- Enable superbeans@superbeans-local plugin
- Remove superbeans hooks (now provided by plugin)
```