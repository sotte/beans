---
# beans-i7lg
title: 'Design Doc: Superbeans Claude Plugin'
status: completed
type: task
priority: normal
tags:
    - artifact:design
    - needs-review
created_at: 2026-01-04T23:41:53Z
updated_at: 2026-01-04T23:49:21Z
parent: beans-js3n
---

# Design: Superbeans as a Claude Code Plugin

## Overview

Convert the superbeans repository into a Claude Code plugin that packages:
- 4 workflow skills (research, brainstorm, plan, execute)
- 4 command wrappers (invoke skills via /superbeans:*-cmd)
- 1 hook (tmux state tracking)

## Plugin Architecture

### Directory Structure

```
superbeans/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── commands/                    # Thin command wrappers
│   ├── research-cmd.md
│   ├── brainstorm-cmd.md
│   ├── plan-cmd.md
│   └── execute-cmd.md
├── skills/                      # Full skill definitions
│   ├── research/SKILL.md
│   ├── brainstorm/SKILL.md
│   ├── plan/SKILL.md
│   └── execute/SKILL.md
├── hooks/
│   ├── hooks.json               # Hook event configuration
│   └── superbeans-track-state.sh
├── .claude/                     # Keep for local development
├── README.md                    # Updated with plugin docs
└── docs/                        # Updated if needed
```

### Plugin Manifest

`.claude-plugin/plugin.json`:
```json
{
  "name": "superbeans",
  "description": "Agentic-first issue tracker with Claude Code workflow integration",
  "version": "0.1.0",
  "author": { "name": "Stefan" },
  "license": "MIT",
  "keywords": ["beans", "issue-tracker", "workflows", "planning"]
}
```

## Skills Design

### Naming Convention

| Original (dotfiles) | Skill Name | Command |
|---------------------|------------|---------|
| bean-research | superbeans:research | /superbeans:research-cmd |
| bean-brainstorm | superbeans:brainstorm | /superbeans:brainstorm-cmd |
| bean-write-plan | superbeans:plan | /superbeans:plan-cmd |
| bean-execute-plan | superbeans:execute | /superbeans:execute-cmd |

### Skill Content

Skills are moved from `~/dotfiles/dotfiles/claude/commands/bean-*.md` with minimal changes:
- Update frontmatter to skill format (name, description)
- Update any internal references to use new skill names

### Command Wrappers

Each command is a thin wrapper following superpowers pattern:

```markdown
---
description: "Research codebase context for a feature, storing findings in a research bean."
---

Invoke the superbeans:research skill and follow it exactly as presented to you
```

## Hooks Design

### Hook Configuration

`hooks/hooks.json`:
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

### Hook Script

Updated `superbeans-track-state.sh` with tmux auto-detection:

```bash
#!/usr/bin/env bash
# Superbeans session state tracker
# Updates @claude_state tmux option based on Claude Code events
# Silently no-ops if tmux is not available

command -v tmux >/dev/null || exit 0

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

## Dependencies

**Required:** superpowers plugin must be installed.

Skills reference these superpowers skills:
- `superpowers:brainstorming` (used by brainstorm)
- `superpowers:writing-plans` (used by plan)
- `superpowers:executing-plans` (used by execute)
- `superpowers:subagent-driven-development` (used by execute)
- `superpowers:finishing-a-development-branch` (used by execute)

This will be documented in README.md.

## Documentation Updates

### README.md

Add section covering:
- Plugin installation: `claude --plugin-dir /path/to/superbeans`
- Available commands and skills
- Workflow overview (research → brainstorm → plan → execute)
- Dependencies (superpowers plugin)
- Hook behavior (tmux state tracking)

### docs/

Review and update if any docs reference the old command structure.

## Testing

1. Local testing: `claude --plugin-dir .`
2. Verify each command invokes its skill correctly
3. Verify hook fires on PreToolUse and Stop events
4. Verify tmux auto-detection works (no errors when tmux unavailable)

## Migration Notes

After this is implemented:
- Remove bean-* commands from ~/dotfiles/dotfiles/claude/commands/
- Remove superbeans-track-state.sh from ~/dotfiles/dotfiles/claude/hooks/
- Update ~/.claude/settings.json to remove old hook references
- Enable superbeans plugin instead