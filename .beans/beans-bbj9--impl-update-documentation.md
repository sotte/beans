---
# beans-bbj9
title: 'Impl: Update documentation'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-05T00:52:04Z
updated_at: 2026-01-05T03:01:45Z
parent: beans-1xgc
---

# Impl: Update documentation

**Files:**
- Modify: `docs/visual-language.md` (add Session State Indicators section)
- Modify: `docs/tmux-integration.md` (update hook script and state table)

---

### Step 1: Add Session State Indicators section to visual-language.md

After the "Blocking/Session Indicators" section (around line 78), add:

```markdown
### Session State Indicators

| State   | Symbol | Color  | Description                        |
|---------|--------|--------|------------------------------------
| Working | `●`    | Green  | Claude actively using tools        |
| Ask     | `?`    | Amber  | Claude asked a question, waiting   |
| Idle    | `⏸`    | Amber  | Session waiting for user input     |
| None    | `○`    | Dim    | No tmux session exists             |
```

### Step 2: Update the state table in tmux-integration.md

Replace the existing table at line 30-35:

```markdown
| Symbol | Color | State | Meaning |
|--------|-------|-------|---------|
| `○ none` | Gray | No session | No tmux session exists for this bean |
| `⏸ idle` | Amber | Idle | Session exists but Claude is waiting for input |
| `? ask` | Amber | Ask | Session exists and Claude asked a question |
| `● work` | Green | Working | Session exists and Claude is actively working |
```

### Step 3: Update the hook script in tmux-integration.md

Replace the hook script at line 46-62:

```bash
#!/usr/bin/env bash
# Superbeans session state tracker
# Updates @claude_state tmux option based on Claude Code events

input=$(cat)
event=$(echo "$input" | jq -r '.hook_event_name')

case "$event" in
  PreToolUse)
    tool_name=$(echo "$input" | jq -r '.tool_name')
    if [ "$tool_name" = "AskUserQuestion" ]; then
      # Claude is asking a question - mark as ask
      tmux set-option @claude_state "idle:ask" 2>/dev/null
    else
      # Claude is about to use a tool - mark as working
      tmux set-option @claude_state working 2>/dev/null
    fi
    ;;
  Stop)
    # Claude finished and is waiting for input - mark as idle
    tmux set-option @claude_state idle 2>/dev/null
    ;;
esac
```

### Step 4: Review changes

Run: `git diff docs/`
Expected: See the documentation updates

### Step 5: Commit

```bash
git add docs/visual-language.md docs/tmux-integration.md
git commit -m "docs: add SessionAsk state to visual-language and tmux-integration

- Add Session State Indicators section to visual-language.md
- Update state table in tmux-integration.md with ask state
- Update hook script to detect AskUserQuestion and set idle:ask

Refs: beans-1xgc"
```
