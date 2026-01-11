---
# beans-gbf1
title: 'Impl: Documentation'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:47:26Z
updated_at: 2026-01-05T00:55:38Z
parent: beans-js3n
blocking:
    - beans-t4iz
---

# Phase 5: Documentation

## Files

- Modify: `README.md`

## Notes on Existing .claude/ Directory

The repo has `.claude/commands/generate-release-notes.md` and `.claude/skills/bubbletea/`. These are for **local development of this repo** and are separate from the plugin structure at root. They coexist:

- `commands/` and `skills/` at root = plugin (distributed to users)
- `.claude/commands/` and `.claude/skills/` = local dev helpers (not distributed)

No action needed - just documenting this is intentional.

## Steps

**Step 1: Read current README**

Read `README.md` to understand current structure.

**Step 2: Add Claude Plugin section**

Add a new section to README.md (after existing content or in appropriate location):

```markdown
## Claude Code Plugin

Superbeans includes a Claude Code plugin for AI-assisted workflow management.

### Installation

```bash
# Option 1: Use plugin directory flag
claude --plugin-dir /path/to/superbeans

# Option 2: Add to enabled plugins in ~/.claude/settings.json
```

### Dependencies

The plugin requires the [superpowers](https://github.com/obra/superpowers) plugin to be installed.

### Available Commands

| Command | Description |
|---------|-------------|
| `/superbeans:research-cmd` | Research codebase context, store in research bean |
| `/superbeans:brainstorm-cmd` | Brainstorm feature design, store in design bean |
| `/superbeans:plan-cmd` | Create implementation plan with task beans |
| `/superbeans:execute-cmd` | Execute tasks from beans |

### Workflow

1. **Research** (optional): Gather codebase context → `/superbeans:research-cmd <bean-id>`
2. **Brainstorm**: Design the feature → `/superbeans:brainstorm-cmd <bean-id>`
3. **Plan**: Break into implementation tasks → `/superbeans:plan-cmd <design-bean-id>`
4. **Execute**: Implement tasks one by one → `/superbeans:execute-cmd <parent-bean-id>`

### Hooks

The plugin includes a tmux state tracking hook that updates `@claude_state`:
- `working` when Claude is using tools
- `idle` when Claude is waiting for input

This is auto-detected and silently disabled if tmux or jq is not available.
```

**Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add Claude plugin documentation

- Installation instructions
- Available commands and workflow
- Dependencies (superpowers plugin)
- Hook behavior description

Refs: beans-js3n"
```