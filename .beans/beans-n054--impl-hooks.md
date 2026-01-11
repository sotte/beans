---
# beans-n054
title: 'Impl: Hooks'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:47:24Z
updated_at: 2026-01-05T00:54:58Z
parent: beans-js3n
blocking:
    - beans-t4iz
---

# Phase 4: Hooks

## Files

- Create: `hooks/hooks.json`
- Create: `hooks/superbeans-track-state.sh`

## Steps

**Step 1: Create hooks directory**

```bash
mkdir -p hooks
```

**Step 2: Create hooks.json**

Create `hooks/hooks.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/hooks/superbeans-track-state.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/hooks/superbeans-track-state.sh"
          }
        ]
      }
    ]
  }
}
```

**Step 3: Create track-state script**

Create `hooks/superbeans-track-state.sh`:

```bash
#!/usr/bin/env bash
# Superbeans session state tracker
# Updates @claude_state tmux option based on Claude Code events
# Silently no-ops if tmux or jq is not available

command -v tmux >/dev/null || exit 0
command -v jq >/dev/null || exit 0

event=$(cat | jq -r '.hook_event_name')

case "$event" in
  PreToolUse)
    tmux set-option @claude_state working 2>/dev/null
    ;;
  Stop)
    tmux set-option @claude_state idle 2>/dev/null
    ;;
esac
```

**Step 4: Make script executable**

```bash
chmod +x hooks/superbeans-track-state.sh
```

**Step 5: Commit**

```bash
git add hooks/
git commit -m "feat: add tmux state tracking hook

- PreToolUse: sets @claude_state to 'working'
- Stop: sets @claude_state to 'idle'
- Auto-detects tmux and jq (silent no-op if unavailable)

Refs: beans-js3n"
```