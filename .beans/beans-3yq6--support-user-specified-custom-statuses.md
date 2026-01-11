---
# beans-3yq6
title: Support user-specified custom statuses
status: scrapped
type: feature
priority: normal
created_at: 2025-12-31T18:06:40Z
updated_at: 2026-01-02T23:51:53Z
---

## Motivation

Different teams and workflows have different needs for tracking work states:

- **Kanban teams** may want: `backlog`, `ready`, `in-progress`, `review`, `done`
- **Agile teams** may need: `icebox`, `ready`, `in-sprint`, `blocked`, `done`
- **Agentic workflows** (Claude Code, etc) may want: `idea`, `designing`, `spec-ready`, `implementing`, `review`, `completed`
- **Content teams** might use: `draft`, `editing`, `review`, `published`

Currently, beans has **hardcoded statuses** that cannot be customized:
- `in-progress` - Currently being worked on
- `todo` - Ready to be worked on  
- `draft` - Needs refinement before it can be worked on
- `completed` - Finished successfully
- `scrapped` - Will not be done

This limits beans' usefulness for teams with specific workflow requirements.

## Current Implementation

Statuses are defined in `internal/config/config.go`:

```go
var DefaultStatuses = []StatusConfig{
    {Name: "in-progress", Color: "yellow", Description: "Currently being worked on"},
    {Name: "todo", Color: "green", Description: "Ready to be worked on"},
    {Name: "draft", Color: "blue", Description: "Needs refinement before it can be worked on"},
    {Name: "completed", Color: "gray", Archive: true, Description: "Finished successfully"},
    {Name: "scrapped", Color: "gray", Archive: true, Description: "Will not be done"},
}
```

Each status has:
- **Name**: The status identifier (used in CLI, frontmatter, filters)
- **Color**: Display color in TUI/CLI output
- **Archive**: Whether beans with this status should be archived
- **Description**: Human-readable description

The comment explicitly states: "Statuses are not configurable - they are hardcoded like types."

## Proposed Extension

Allow users to define custom statuses in `.beans.yml`:

```yaml
beans:
  path: .beans
  default_status: backlog

  statuses:
    - name: backlog
      color: gray
      description: "Work not yet prioritized"
    - name: ready
      color: green
      description: "Ready to be picked up"
    - name: in-progress
      color: yellow
      description: "Currently being worked on"
    - name: review
      color: purple
      description: "Awaiting review"
    - name: done
      color: gray
      archive: true
      description: "Completed"
```

### Key Design Considerations

1. **Backwards compatibility**: If no `statuses` key in config, use DefaultStatuses
2. **Status order matters**: Order in config determines sort priority in lists/TUI
3. **Archive flag**: At least one status should be marked `archive: true` for cleanup
4. **Validation**: Warn if a bean has an unknown status (but don't fail - allows migration)
5. **TUI support**: Status picker should show configured statuses
6. **GraphQL schema**: Status enum may need to become a string field, or use dynamic validation

### Migration Path

- Existing beans with default statuses continue to work
- Users can add custom statuses alongside defaults
- Full replacement of default statuses is opt-in