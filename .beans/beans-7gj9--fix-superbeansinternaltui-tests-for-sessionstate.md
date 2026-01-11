---
# beans-7gj9
title: Fix superbeans/internal/tui tests for SessionState
status: todo
type: bug
priority: low
created_at: 2026-01-02T22:12:55Z
updated_at: 2026-01-02T22:12:55Z
---

Tests in feature_test.go use old map[string]bool signature but code now uses map[string]SessionState. Need to update tests to match.