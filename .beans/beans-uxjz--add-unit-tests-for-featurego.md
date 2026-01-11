---
# beans-uxjz
title: Add unit tests for feature.go
status: completed
type: task
priority: high
tags:
    - artifact:impl
created_at: 2026-01-01T22:00:25Z
updated_at: 2026-01-01T22:07:07Z
parent: beans-kwuv
---

## Problem

The helper functions in `feature.go` are untested. These are pure functions that would benefit from unit tests.

## File
`superbeans/internal/tui/feature.go`

## Functions to Test
- `HasTag(b *bean.Bean, tag string) bool`
- `HasArtifactTag(b *bean.Bean) bool`
- `GetArtifactType(b *bean.Bean) string`
- `HasNeedsReview(b *bean.Bean) bool`
- `NewFeatureItem(feature *bean.Bean, children []*bean.Bean, sessions map[string]bool) FeatureItem`

## Why It Matters
These functions are core to the feature display logic. Bugs here would cause incorrect phase/review display.

## Test Cases to Cover
- `HasTag`: tag present, tag absent, nil tags slice
- `GetArtifactType`: each artifact type (research, design, plan, impl), no artifact tag, multiple tags
- `HasNeedsReview`: with tag, without tag
- `NewFeatureItem`: with/without children, with/without sessions, needs-review bubbling up from children