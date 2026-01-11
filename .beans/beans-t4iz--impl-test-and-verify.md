---
# beans-t4iz
title: 'Impl: Test and verify'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:47:28Z
updated_at: 2026-01-05T00:56:31Z
parent: beans-js3n
---

# Phase 6: Test and Verify

## Steps

**Step 1: Verify plugin structure**

```bash
ls -la .claude-plugin/
ls -la commands/
ls -la skills/
ls -la hooks/
```

Expected: All directories exist with correct files.

**Step 2: Test plugin loading**

```bash
claude --plugin-dir . --help
```

Expected: No errors about plugin loading.

**Step 3: Test commands are registered**

Start claude with plugin and check that commands appear:
- `/superbeans:research-cmd`
- `/superbeans:brainstorm-cmd`
- `/superbeans:plan-cmd`
- `/superbeans:execute-cmd`

**Step 4: Test hook script standalone**

```bash
echo '{"hook_event_name": "PreToolUse"}' | ./hooks/superbeans-track-state.sh
echo $?
```

Expected: Exit 0 (or silent if tmux not available).

**Step 5: Verify skill content**

Quick read of each skill to ensure content transferred correctly:

```bash
head -20 skills/research/SKILL.md
head -20 skills/brainstorm/SKILL.md
head -20 skills/plan/SKILL.md
head -20 skills/execute/SKILL.md
```

Expected: Each has correct frontmatter (name, description) and content.

**Step 6: Final commit (if any fixes needed)**

```bash
git status
# If any fixes were made:
git add -A
git commit -m "fix: address plugin verification issues

Refs: beans-js3n"
```