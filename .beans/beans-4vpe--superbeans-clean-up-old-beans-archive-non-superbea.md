---
# beans-4vpe
title: 'SUPERBEANS: clean up old beans, archive non-superbeans beans'
status: todo
type: task
created_at: 2026-01-02T19:09:54Z
updated_at: 2026-01-02T19:09:54Z
---

## Goal

This repo is being forked for superbeans. Old beans that are specific to the original beans project should be archived/scrapped to keep focus on superbeans work.

## Beans to Remove (scrap or archive)

### Feature beans (non-SUPERBEANS)

| ID | Status | Title | Recommendation |
|----|--------|-------|----------------|
| beans-mmyp | todo | Workflow CLI commands | scrap - beans-specific |
| beans-hz87 | todo | Add blocked-by relationship (possibly replacing blocking) | scrap - beans-specific |
| beans-l3jg | todo | Add tag editing UI to TUI | scrap - beans-specific |
| beans-1mal | todo | Expand beans prime with agent-specific output | **keep** - relates to beans-erio |
| beans-bgpd | todo | Represent scrapped beans in superbeans | **keep** - SUPERBEANS related |
| beans-3490 | todo | Show priority in superbeans | **keep** - SUPERBEANS related |
| beans-8olg | draft | Add --unblocked filter to find actionable beans | scrap - beans-specific |
| beans-iggk | draft | Add beans graph command for relationship visualization | scrap - beans-specific |
| beans-ikw9 | draft | beans release - Release management integration | scrap - beans-specific |
| beans-lbjp | draft | beans web - Web UI server | scrap - beans-specific |
| beans-wxl4 | draft | Experiment with integrating agent UI into web UI using ACP | scrap - beans-specific |
| beans-6hbt | draft | Generate embeddings for beans using OpenAI | scrap - beans-specific |
| beans-8rr2 | draft | Quick bean properties popup (status, tags, artifact type) | scrap - beans-specific |
| beans-j03e | draft | Show relationship counts in list output | scrap - beans-specific |
| beans-qr4m | draft | Superbeans v2: Agent-first workflow system redesign | **keep** - SUPERBEANS roadmap |
| beans-3yq6 | draft | Support user-specified custom statuses | scrap - beans-specific |
| beans-jwy7 | draft | Use semantic markdown sections in bean bodies | scrap - beans-specific |
| beans-kp3h | draft | Wrap beans in an MCP server | scrap - beans-specific |

### Summary

**Scrap (15 beans):**
- beans-mmyp, beans-hz87, beans-l3jg, beans-8olg, beans-iggk, beans-ikw9
- beans-lbjp, beans-wxl4, beans-6hbt, beans-8rr2, beans-j03e, beans-3yq6
- beans-jwy7, beans-kp3h

**Keep (6 beans):**
- beans-1mal (agent-specific prime output)
- beans-bgpd (scrapped beans in superbeans)
- beans-3490 (priority in superbeans)
- beans-qr4m (superbeans v2 roadmap)
- Plus all beans prefixed with "SUPERBEANS:"

## Tasks

- [ ] Review this list with Stefan
- [ ] Scrap confirmed beans
- [ ] Run `beans archive` to clean up