---
# beans-lb0j
title: 'SUPERBEANS: Add idle/working session indicator'
status: completed
type: feature
priority: high
created_at: 2026-01-02T15:39:09Z
updated_at: 2026-01-02T21:06:07Z
---

Show whether Claude sessions need human input (idle) or are actively working. Uses Claude Code hooks to track state.

## Motivation

When running multiple Claude agents in tmux sessions, the key question is: **"Which sessions need me to respond?"**

Currently superbeans shows:
- ✓ Whether a session exists (`● live` / `○ none`)
- ✗ Whether the session is waiting for input or actively working

This means Stefan has to attach to each session to check if Claude asked a question. With 3-4 concurrent agents, this is friction.

**Goal:** At a glance, know which sessions need human attention.

## Requirements

1. Detect when Claude is waiting for user input (idle) vs actively working
2. Store this state somewhere accessible to superbeans
3. Display in superbeans TUI with clear visual distinction
4. Automatic cleanup when session ends

## Implementation Options

### Option A: tmux session options (Recommended)

**Mechanism:** Use tmux's user-defined options per session

**Hook script** (`~/.claude/hooks/track-state.sh`):
```bash
#!/bin/bash
event=$(cat | jq -r '.hook_event_name')
case "$event" in
  PreToolUse)  tmux set-option @claude_state working 2>/dev/null ;;
  Stop)        tmux set-option @claude_state idle 2>/dev/null ;;
esac
```

**Settings** (`.claude/settings.json`):
```json
{
  "hooks": {
    "PreToolUse": [{"matcher": "*", "hooks": [{"type": "command", "command": "~/.claude/hooks/track-state.sh"}]}],
    "Stop": [{"hooks": [{"type": "command", "command": "~/.claude/hooks/track-state.sh"}]}]
  }
}
```

**Superbeans query:**
```bash
tmux show-option -t <session> -v @claude_state
```

**Pros:**
- Simple implementation
- State dies with session (automatic cleanup)
- Hook doesn't need to know bean ID
- No file management
- No concurrency issues

**Cons:**
- State not inspectable outside tmux
- Limited to string values

---

### Option B: Shared JSON file

**Mechanism:** All hooks write to `~/.claude/agent-states.json`

**Hook script:**
```bash
#!/bin/bash
event=$(cat | jq -r '.hook_event_name')
session=$(tmux display-message -p '#S' 2>/dev/null)
state_file=~/.claude/agent-states.json

[ -z "$session" ] && exit 0

case "$event" in
  PreToolUse)  state="working" ;;
  Stop)        state="idle" ;;
  *)           exit 0 ;;
esac

# Atomic update with flock
(
  flock 200
  current=$(cat "$state_file" 2>/dev/null || echo '{}')
  echo "$current" | jq --arg s "$session" --arg st "$state" '.[$s] = $st' > "$state_file.tmp"
  mv "$state_file.tmp" "$state_file"
) 200>"$state_file.lock"
```

**Superbeans:** Parse JSON file

**Pros:**
- Inspectable with `cat`
- Extensible (could add timestamps, last tool, etc.)
- Works outside tmux context

**Cons:**
- Needs file locking for concurrency
- Requires cleanup of stale entries
- More complex hook script
- jq dependency

---

### Option C: tmux window title

**Mechanism:** Encode state in the tmux window name

**Hook script:**
```bash
#!/bin/bash
event=$(cat | jq -r '.hook_event_name')
window=$(tmux display-message -p '#W' 2>/dev/null)

[ -z "$window" ] && exit 0

# Strip existing state prefix
base_name=$(echo "$window" | sed 's/^\[.*\] //')

case "$event" in
  PreToolUse)  tmux rename-window "[●] $base_name" ;;
  Stop)        tmux rename-window "[◐] $base_name" ;;
esac
```

**Superbeans:** Parse window names from `tmux list-windows`

**Pros:**
- **Visible directly in tmux UI** (status bar shows state)
- Automatic cleanup
- No extra queries needed

**Cons:**
- Pollutes window names
- Parsing is fragile
- Limited to what fits in window name

---

## Comparison

| Aspect | A: tmux option | B: JSON file | C: window title |
|--------|----------------|--------------|-----------------|
| **Complexity** | Simple | Medium | Simple |
| **Cleanup** | Automatic | Manual | Automatic |
| **Concurrency** | Safe | Needs locking | Safe |
| **Inspectable** | `tmux show-option` | `cat file` | Visible in tmux |
| **Extensibility** | Limited | Good | Limited |
| **Dependencies** | tmux only | jq, flock | tmux only |
| **Visual feedback** | None | None | **In tmux UI** |

## Recommendation

**Option A (tmux session options)** is the cleanest for programmatic access.

**Option C (window title)** has the bonus of being visible in tmux itself without needing superbeans.

Could even combine: use Option A for superbeans, and optionally also update window title for direct tmux visibility.

## UI Design

Current: `● live` / `○ none`

Implemented:
- `○ none` - no session (gray)
- `● work` - working (green)
- `⏸ idle` - idle/waiting (amber) ← **needs attention**

The amber color draws the eye to sessions needing input.

## Implementation Tasks

- [x] Update `session.go` to query state via `@claude_state` tmux option
- [x] Update `features.go` display with three-state indicator
- [x] Update `detail.go` display with session state in header
- [x] Add color styling for states (green=working, amber=idle, gray=none)
- [x] Add `docs/tmux-integration.md` with hook setup instructions
- [x] Create hook script at `~/dotfiles/dotfiles/claude/hooks/superbeans-track-state.sh`
- [x] Enable hooks in `~/dotfiles/dotfiles/claude/settings.json`
- [ ] Test with multiple concurrent sessions (manual testing needed)

## Current Status

**Ready for testing.** All code is committed:
- Commit: `631e985` - feat(superbeans): add idle/working session state indicator
- Hook installed and enabled in dotfiles

**To test:**
1. Restart Claude Code (or start new session) to pick up hook changes
2. Run `superbeans` in another terminal
3. Verify state changes: `tmux show-option -v @claude_state`

## Related

- [[beans-qr4m]] - Superbeans v2 long-term thinking
- Claude Code hooks documentation