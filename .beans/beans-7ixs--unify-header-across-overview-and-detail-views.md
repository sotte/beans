---
# beans-7ixs
title: Unify header across overview and detail views
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T21:45:09Z
updated_at: 2026-01-01T21:46:09Z
parent: beans-kwuv
---

Standardize the header layout across both views:

**Current:**
- Overview: Just 'SUPERBEANS' in purple
- Detail: '← beans-id  Title' with session info

**New layout (both views):**
```
SUPERBEANS  [optional: → Feature Title]
────────────────────────────────────────
```

- Always show 'SUPERBEANS' brand in purple
- In detail view, append feature title (dimmed or different color)
- Add separator line below header (like detail view has)
- Use consistent colors from styles.go