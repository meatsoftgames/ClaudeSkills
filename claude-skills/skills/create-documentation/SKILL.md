---
name: create-documentation
description: Use when starting a new project that needs the standard MeatSoft doc scaffolding, or when end-project detects missing docs. Creates CLAUDE.md, README.md, docs/HANDOFF.md, and docs/ subfolders for devlog, decisions, specs, plans, documentation, plus tasks.md, FAQ.md, gotchas.md. Idempotent — won't overwrite existing files.
user_invocable: true
---

# Create Documentation

Bootstrap the standard repo doc structure for a project. Idempotent — safe to run on existing repos with partial structure.

## Structure created

```
<repo>/
├── CLAUDE.md                    ← session-start orientation
├── README.md                    ← project status
└── docs/
    ├── HANDOFF.md               ← live next-session resume point
    ├── devlog/                  ← one .md per session
    ├── decisions/               ← one .md per decision (ADR-style)
    ├── specs/                   ← one .md per spec (what to build)
    │   └── archive/             ← shipped/superseded specs
    ├── plans/                   ← one .md per plan (how to build it)
    │   └── archive/             ← shipped/superseded plans
    ├── documentation/           ← user-facing guides + code-derived references
    ├── tasks.md                 ← project backlog
    ├── FAQ.md                   ← project Q&A
    └── gotchas.md               ← numbered list of post-hoc gotchas
```

## How to run

1. **Check what exists.** For each path, only create what's missing. Don't overwrite. Track what was found vs. created for the summary.
2. **Create folders** with `mkdir -p`.
3. **Stub each missing file** with the templates below. Empty folders that need to be tracked by git get a `README.md` describing what goes in them.
4. **Don't commit.** The user reviews; end-project commits as part of its sweep when called from there.

## File templates

### CLAUDE.md
```
# CLAUDE.md

Session-start orientation for this project.

## Read in order

When resuming, read these files (in this order):
1. README.md
2. docs/HANDOFF.md
3. Most recent file in docs/devlog/

## Tone

(project-specific preferences)

## Project-specific docs

(list any project-specific docs end-project should keep current — e.g. architecture plans, phase docs)
```

### README.md
```
# <Project Name>

<one-line description>

## Status

_TODO: current status_

## Quick start

_TODO: setup instructions_
```

### docs/HANDOFF.md
```
# Handoff

_No session has ended yet. Run /claude-skills:end-project after a session to populate this file._
```

### docs/devlog/README.md
```
# Devlog

One markdown file per session, named `YYYY-MM-DD-session.md` or `session-N.md`. Append-only — past sessions stay as-is.
```

### docs/decisions/README.md
```
# Decisions

One markdown file per architectural decision (ADR-style). Title = the decision, body = motive + alternatives considered.
```

### docs/specs/README.md
```
# Specs

What we're building. One markdown file per spec. Move to `archive/` when shipped or superseded.
```

### docs/specs/archive/README.md
```
# Archive

Specs that are shipped or superseded.
```

### docs/plans/README.md
```
# Plans

How we're building it. One markdown file per plan. Move to `archive/` when shipped or superseded.
```

### docs/plans/archive/README.md
```
# Archive

Plans that are shipped or superseded.
```

### docs/documentation/README.md
```
# Documentation

User-facing guides and code-derived references. Organise by topic.
```

### docs/tasks.md
```
# Tasks

Project backlog.

## To do
- [ ] _add tasks here_

## In progress

## Done
```

### docs/FAQ.md
```
# FAQ

_No questions yet._
```

### docs/gotchas.md
```
# Gotchas

Numbered list of post-hoc "we got bitten by X" notes.

Format: short title, what bit us, the fix, and a `See:` pointer to the commit or file.
```

## Final summary

Show:
- What was already there (skipped)
- What was created
- Any unexpected partial structure

Then stop.
