---
# beans-i5ru
title: Include parent feature in detail view list (not header)
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T21:42:02Z
updated_at: 2026-01-01T21:44:10Z
parent: beans-kwuv
---

Currently the parent feature is shown in the header of the detail view, but pressing enter on it doesn't work (enter only works on child beans in the list).

Solution: Move the parent feature into the list as the first item, so:
- It can be selected with cursor navigation
- Enter opens it in the editor like any other bean
- Yank works on it
- Consistent UX with child beans

The header should still show the feature title for context, but the actionable row should be in the list.