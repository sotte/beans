---
# beans-0eti
title: 'Impl: Add beans prime hooks to plugin'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-07T20:26:54Z
updated_at: 2026-01-07T20:29:56Z
parent: beans-2wjr
---

# Phase 2: Add beans prime hooks to superbeans plugin

## Files to Modify

1. **Update**: `/home/stefan/projects/superbeans/hooks/hooks.json`

## Steps

### Step 1: Update hooks.json

**File**: `/home/stefan/projects/superbeans/hooks/hooks.json`

Add SessionStart and PreCompact hooks for `beans prime`. Final content:
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          { "type": "command", "command": "beans prime" }
        ]
      }
    ],
    "PreCompact": [
      {
        "hooks": [
          { "type": "command", "command": "beans prime" }
        ]
      }
    ],
    "PreToolUse": [
      {
        "hooks": [
          { "type": "command", "command": "${CLAUDE_PLUGIN_ROOT}/hooks/superbeans-track-state.sh" }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "${CLAUDE_PLUGIN_ROOT}/hooks/superbeans-track-state.sh" }
        ]
      }
    ]
  }
}
```

## Commit Message
```
feat: add beans prime hooks to superbeans plugin

SessionStart and PreCompact hooks now run `beans prime` to inject
beans context into Claude Code sessions.

Refs: beans-2wjr
```