---
# beans-5w3x
title: 'Impl: Superbeans repo cleanup'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-07T20:26:38Z
updated_at: 2026-01-07T20:29:39Z
parent: beans-2wjr
---

# Phase 1: Superbeans Repo Cleanup

## Files to Modify

1. **Delete**: `/home/stefan/projects/superbeans/extras/` (entire directory)
2. **Update**: `/home/stefan/projects/superbeans/.claude-plugin/marketplace.json`

## Steps

### Step 1: Delete outdated extras folder
```bash
rm -rf /home/stefan/projects/superbeans/extras/
```

### Step 2: Update marketplace.json

**File**: `/home/stefan/projects/superbeans/.claude-plugin/marketplace.json`

Replace contents with:
```json
{
  "name": "superbeans",
  "owner": {
    "name": "Stefan"
  },
  "plugins": [
    {
      "name": "superbeans",
      "source": ".",
      "description": "Agentic-first issue tracker with Claude Code workflow integration",
      "version": "0.1.0"
    }
  ]
}
```

## Commit Message
```
chore: clean up superbeans plugin structure

- Remove outdated extras/ directory with old beans-prime plugin
- Update marketplace.json to point to superbeans plugin at repo root

Refs: beans-2wjr
```