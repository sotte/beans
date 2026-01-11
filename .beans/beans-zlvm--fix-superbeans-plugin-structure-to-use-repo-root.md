---
# beans-zlvm
title: Fix superbeans plugin structure to use repo root
status: completed
type: task
priority: normal
created_at: 2026-01-07T21:12:44Z
updated_at: 2026-01-07T21:14:24Z
parent: beans-2wjr
---

# Fix Superbeans Plugin Structure

## Resolution

Fixed plugin structure to match superpowers reference implementation:
- Changed marketplace.json `source` from `./plugins/superbeans` to `./`
- Deleted unnecessary `plugins/` folder
- commands/, skills/, hooks/ remain at repo root

## Final Structure

```
superbeans/
├── .claude-plugin/
│   ├── marketplace.json    # source: "./"
│   └── plugin.json
├── commands/               # At root
├── skills/                 # At root
└── hooks/                  # At root
```

## How It Works

1. Marketplace registered via CLI → stored in ~/.claude/plugins/installed_plugins.json
2. marketplace.json points to repo root with `"source": "./"`
3. Plugin content (commands, skills, hooks) loaded from repo root
4. No `extraKnownMarketplaces` needed in settings.json