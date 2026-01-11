---
# beans-js3n
title: 'Refactor: pull in bean-* commands into the repo, turn commands into skills, offer bean-*-cmd to execute the skill'
status: completed
type: feature
priority: normal
created_at: 2026-01-04T23:07:08Z
updated_at: 2026-01-05T02:56:49Z
---

Convert superbeans repo into a Claude Code plugin, pulling in the bean-* workflow commands from dotfiles.

## Merged From

- beans-p7ho (scrapped): bundling/distribution question → answered by plugin architecture

## Motivation

Currently bean-* commands live in ~/dotfiles and the hook is manually installed. Converting to a Claude Code plugin:
- Single installation via plugin system
- Skills/commands/hooks packaged together
- Easy updates
- Better maintainability

## Components to Include

### Skills (from ~/dotfiles/dotfiles/claude/commands/)
- bean-research → superbeans:research
- bean-brainstorm → superbeans:brainstorm  
- bean-write-plan → superbeans:plan
- bean-execute-plan → superbeans:execute

### Commands
- research-cmd.md → invokes superbeans:research skill
- brainstorm-cmd.md → invokes superbeans:brainstorm skill
- plan-cmd.md → invokes superbeans:plan skill
- execute-cmd.md → invokes superbeans:execute skill

### Hooks
- superbeans-track-state.sh (PreToolUse + Stop events, auto-detects tmux)

## Plugin Structure

superbeans/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── research-cmd.md
│   ├── brainstorm-cmd.md
│   ├── plan-cmd.md
│   └── execute-cmd.md
├── skills/
│   ├── research/SKILL.md
│   ├── brainstorm/SKILL.md
│   ├── plan/SKILL.md
│   └── execute/SKILL.md
├── hooks/
│   ├── hooks.json
│   └── superbeans-track-state.sh
└── README.md (updated with plugin docs)

## Dependencies

- Assumes superpowers plugin is installed (skills reference superpowers:* skills)

## Tasks

- [ ] Create .claude-plugin/plugin.json
- [ ] Create skills/ directory with 4 skills
- [ ] Create commands/ directory with 4 -cmd wrappers
- [ ] Create hooks/ directory with hooks.json and track-state script
- [ ] Update README.md with plugin installation/usage
- [ ] Update docs/ if needed
- [ ] Test with `claude --plugin-dir .`