---
# beans-q5im
title: Add 'create feature bean' to superbeans TUI
status: draft
type: feature
created_at: 2026-01-04T23:15:07Z
updated_at: 2026-01-04T23:15:07Z
---

Add the ability to create a new feature bean directly from the superbeans TUI.

## Motivation

Currently you have to exit the TUI to create new beans via CLI. Adding a create action would streamline the workflow.

## Tasks

- [ ] Add key binding (e.g., 'c' or 'n') for create action
- [ ] Implement create modal/form for entering bean details
- [ ] Call beans create via GraphQL mutation
- [ ] Refresh view after creation
- [ ] Update footer hints