---
# beans-19cx
title: 'Impl: Delete old superbeans cruft from dotfiles'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-07T20:27:10Z
updated_at: 2026-01-07T20:30:43Z
parent: beans-2wjr
---

# Phase 4: Delete old superbeans cruft from dotfiles

## Files to Delete

### Commands (replaced by superbeans skills)
- `~/dotfiles/dotfiles/claude/commands/bean-research.md`
- `~/dotfiles/dotfiles/claude/commands/bean-brainstorm.md`
- `~/dotfiles/dotfiles/claude/commands/bean-write-plan.md`
- `~/dotfiles/dotfiles/claude/commands/bean-execute-plan.md`

### Docs (now lives with plugin)
- `~/dotfiles/dotfiles/claude/docs/SUPERBEANS.md`

### Hooks (now provided by plugin)
- `~/dotfiles/dotfiles/claude/hooks/superbeans-track-state.sh`

## Steps

```bash
rm ~/dotfiles/dotfiles/claude/commands/bean-research.md
rm ~/dotfiles/dotfiles/claude/commands/bean-brainstorm.md
rm ~/dotfiles/dotfiles/claude/commands/bean-write-plan.md
rm ~/dotfiles/dotfiles/claude/commands/bean-execute-plan.md
rm ~/dotfiles/dotfiles/claude/docs/SUPERBEANS.md
rm ~/dotfiles/dotfiles/claude/hooks/superbeans-track-state.sh
```

## Commit Message (in dotfiles repo)
```
chore: remove old superbeans files (now in plugin)

- Delete bean-* commands (replaced by superbeans skills)
- Delete SUPERBEANS.md docs (lives with plugin)
- Delete superbeans-track-state.sh hook (provided by plugin)
```