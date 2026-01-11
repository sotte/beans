---
# beans-g0z1
title: 'Impl: .gitattributes setup'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:40:17Z
updated_at: 2026-01-04T23:46:15Z
parent: beans-ld6n
---

## Phase 1: .gitattributes Setup

**Files:**
- Create: `.gitattributes`

**Step 1: Create .gitattributes file**

```
*.golden -text
```

This prevents git from normalizing line endings in golden files, which would cause CI failures.

**Step 2: Verify file was created**

Run: `cat .gitattributes`
Expected: `*.golden -text`

**Step 3: Commit**

```bash
git add .gitattributes
git commit -m "chore: add .gitattributes for golden file handling

Refs: beans-ld6n"
```
