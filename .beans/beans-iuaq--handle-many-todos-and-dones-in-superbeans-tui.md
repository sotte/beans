---
# beans-iuaq
title: Handle many TODOs and DONEs in superbeans TUI
status: completed
type: feature
priority: normal
created_at: 2026-01-05T15:03:43Z
updated_at: 2026-01-05T19:58:01Z
---

Handle large TODO/DONE lists in superbeans TUI with section limits and dedicated search view.

## Summary

When lists grow large, the current UI has problems with screen space, discoverability, and navigation.

## Solution

Two-part approach:
1. **Section limits** - Show N items per section with '+X more' overflow indicator
2. **Dedicated search view** - Press '/' for full-screen search across all features

See beans-qb4j for detailed design document.