---
# beans-f132
title: 'Impl: Plugin manifest and structure'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:47:18Z
updated_at: 2026-01-05T00:52:03Z
parent: beans-js3n
blocking:
    - beans-t4iz
---

# Phase 1: Plugin Manifest and Structure

## Files

- Create: `.claude-plugin/plugin.json`

## Steps

**Step 1: Create plugin directory**

```bash
mkdir -p .claude-plugin
```

**Step 2: Create plugin.json**

Create `.claude-plugin/plugin.json`:

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

**Step 3: Commit**

```bash
git add .claude-plugin/
git commit -m "feat: add Claude plugin manifest

- Create .claude-plugin/plugin.json
- Plugin name: superbeans
- Version: 0.1.0

Refs: beans-js3n"
```