---
# beans-om3k
title: 'Design Doc: Work indicator ask state'
status: completed
type: task
priority: normal
tags:
    - artifact:design
    - needs-review
created_at: 2026-01-05T00:46:47Z
updated_at: 2026-01-05T00:53:47Z
parent: beans-1xgc
---

# Design: Work Indicator Ask State

## Problem

The work indicator currently shows "working" when `AskUserQuestion` is triggered. It should switch to an "ask" indicator since the user needs to take action.

## Solution

Add a fourth session state `SessionAsk` that is set when Claude calls `AskUserQuestion`. This provides visual feedback that Claude is waiting for user input on a specific question.

## State Model

The `@claude_state` tmux option will use a format that encodes the reason:

| tmux value  | SessionState   | Display  | Symbol | Color    |
|-------------|----------------|----------|--------|----------|
| `working`   | SessionWorking | `work`   | `●`    | Green    |
| `idle:ask`  | SessionAsk     | `ask`    | `?`    | Amber    |
| `idle`      | SessionIdle    | `idle`   | `○`    | Amber    |
| (none)      | SessionNone    | `none`   | `○`    | Dim gray |

## Hook Logic

The hook script detects `AskUserQuestion` and sets the appropriate state:

```bash
case "$event" in
  PreToolUse)
    tool_name=$(echo "$input" | jq -r '.tool_name')
    if [ "$tool_name" = "AskUserQuestion" ]; then
      tmux set-option @claude_state "idle:ask"
    else
      tmux set-option @claude_state working
    fi
    ;;
  Stop)
    tmux set-option @claude_state idle
    ;;
esac
```

**Behavior:**
1. When any tool fires → set `working`
2. When `AskUserQuestion` specifically fires → set `idle:ask` (overrides working)
3. When `Stop` fires → set `idle` (clears the ask state)

## TUI Changes

### New Constant (`session.go`)

```go
const (
    SessionNone    SessionState = "none"
    SessionIdle    SessionState = "idle"
    SessionAsk     SessionState = "ask"
    SessionWorking SessionState = "working"
)
```

### Parsing Logic (`session.go`)

```go
func querySessionState(sessionName string) SessionState {
    // ... get output ...
    state := strings.TrimSpace(string(output))
    switch {
    case state == "working":
        return SessionWorking
    case state == "idle:ask":
        return SessionAsk
    case state == "idle", state == "":
        return SessionIdle
    default:
        return SessionIdle
    }
}
```

### New Style (`styles.go`)

```go
StyleSessionAsk = lipgloss.NewStyle().
    Foreground(ColorAmber)
```

### Rendering (`features.go`, `detail.go`)

```go
switch item.SessionState {
case SessionWorking:
    sessionStr = "● work"
    sessionStyle = StyleSessionWorking
case SessionAsk:
    sessionStr = "? ask"
    sessionStyle = StyleSessionAsk
case SessionIdle:
    sessionStr = "○ idle"
    sessionStyle = StyleSessionIdle
default:
    sessionStr = "○ none"
    sessionStyle = StyleSessionNone
}
```

## Documentation Updates

Update `docs/visual-language.md` with new Session State Indicators section:

```markdown
### Session State Indicators

| State   | Symbol | Color  | Description                        |
|---------|--------|--------|------------------------------------|
| Working | `●`    | Green  | Claude actively using tools        |
| Ask     | `?`    | Amber  | Claude asked a question, waiting   |
| Idle    | `○`    | Amber  | Session waiting for user input     |
| None    | `○`    | Dim    | No tmux session exists             |
```

## Files to Modify

1. `superbeans/internal/tui/session.go` - Add `SessionAsk` constant, update parsing
2. `superbeans/internal/tui/styles.go` - Add `StyleSessionAsk`
3. `superbeans/internal/tui/features.go` - Update rendering switch
4. `superbeans/internal/tui/detail.go` - Update rendering switch
5. `docs/visual-language.md` - Add Session State Indicators section
6. `docs/tmux-integration.md` - Update hook script example
7. `superbeans/internal/tui/session_test.go` - Add test for SessionAsk
8. `superbeans/internal/tui/feature_test.go` - Add test for SessionAsk state

## Test Plan

- [ ] Unit test: `SessionAsk` constant has correct value
- [ ] Unit test: `querySessionState` parses `idle:ask` correctly
- [ ] Unit test: `NewFeatureItem` correctly sets `SessionAsk` state
- [ ] Manual test: Start session, trigger AskUserQuestion, verify `? ask` appears
- [ ] Manual test: After answering, verify state returns to `○ idle`
