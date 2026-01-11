---
# beans-ipwi
title: 'SUPERBEANS: make doc folder the source of truth, derive implementation from docs'
status: completed
type: feature
priority: normal
created_at: 2026-01-02T19:01:29Z
updated_at: 2026-01-02T19:38:56Z
---

## Concept

Documentation-first development: the `doc/` folder becomes the canonical source of truth for superbeans behavior and design. Implementation follows documentation, not the other way around.

## Structure

```
doc/
├── README.md              # High-level overview, architecture, core concepts
├── features/
│   ├── session-management.md
│   ├── bean-display.md
│   ├── keyboard-navigation.md
│   └── ...
├── configuration.md       # Config file format, options
└── hooks.md               # Claude Code hooks integration
```

## Principles

1. **Docs describe intent** - What superbeans should do, not what it currently does
2. **Implementation follows docs** - When docs and code disagree, update the code
3. **One main doc** - `doc/README.md` gives the 30,000ft view
4. **Detailed docs for details** - Separate files for each feature area

## Benefits

- Easier onboarding for contributors (human or agent)
- Clear specification when implementing new features
- Design decisions are documented before coding
- Agents can read docs to understand expected behavior

## Tasks

- [x] Create `doc/` folder structure
- [x] Write high-level `doc/README.md` with architecture overview
- [ ] Document existing features in separate files (deferred - keeping single file for now)
- [x] Establish convention for doc-first development workflow (based on SUPERBEANS.md)