---
name: end-project
description: Use when the user says "end the session", "wrap up", "end project", "close out the session", or a clear variant. Captures what shipped this session so context can be cleared and a future session can resume cleanly. Writes to repo docs (devlog, decisions, gotchas, HANDOFF, etc.) as the source of truth, then mirrors to Notion as a backup. Calls create-documentation first if the standard doc structure is missing. Don't skip the repo writes.
user_invocable: true
---

# End Project

Document what shipped this session so context can be cleared and the next session resumes cleanly.

## Why this exists

Mid-session the conversation holds the full picture. The moment it ends, all of that evaporates unless it lands in writing. **The repo is the source of truth** (devlog, decisions, gotchas, HANDOFF). **Notion is a reference layer** for when you're away from the project (e.g. on the web app). This skill walks both, repo first.

## Before you start

**Confirm session context is intact.** If compacted or unsure what shipped, ask the user for a quick summary. Fiction in the docs compounds.

**Check git state in parallel:**
- `git log --oneline -20` — commit hashes you'll reference
- `git status` — uncommitted = part of the handoff (or commit first; ask)
- `git diff --stat HEAD~N` or `main` — scope

**Verify doc structure exists.** Check that the following are present:
- `CLAUDE.md`, `README.md`
- `docs/HANDOFF.md`
- `docs/devlog/`, `docs/decisions/`
- `docs/specs/`, `docs/specs/archive/`
- `docs/plans/`, `docs/plans/archive/`
- `docs/documentation/`
- `docs/tasks.md`, `docs/FAQ.md`, `docs/gotchas.md`

If anything is missing, **run the create-documentation skill first** to scaffold what's missing, then continue.

**Build a short list** of what the session did before writing: what changed, commit(s), motive.

## What to update — repo first

### 1. docs/devlog/ (always — new file)

Add a new markdown file: `docs/devlog/YYYY-MM-DD-session.md` (or `session-N.md` if the project uses numbered sessions — check existing files for the pattern).

Structure:

```
# Session — YYYY-MM-DD: <one-line outcome>

**Commits**: `hash1`, `hash2`, …

## What shipped
- Bullet per change. Commit hash in backticks. Motive in one clause.

## What we learned
(only if non-obvious — skip for routine work)

## Decisions
(only if a design choice was made — cross-link to docs/decisions/ files)

## Carryover
(what's pending — bridges to HANDOFF)
```

Match scope. Routine session = 3 bullets under "What shipped" and nothing else.

### 2. docs/decisions/ (if any architectural decisions)

For each architectural decision made this session, add a new file: `docs/decisions/YYYY-MM-DD-<short-slug>.md`.

Structure (ADR-style):

```
# <Decision title>

**Date**: YYYY-MM-DD
**Status**: accepted | superseded by <link> | reverted

## Context

What was the situation that forced a decision.

## Decision

What we're doing.

## Alternatives considered

What else was on the table and why it lost.

## Consequences

What this enables, what it forecloses, what to watch for.
```

### 3. docs/gotchas.md (if any post-hoc gotchas surfaced)

Append numbered entries to `docs/gotchas.md`. Read the file first to find the next free number (#N+1 after the last existing entry).

Format each entry:

```
## #N — Short title

**What bit us**: 1-2 sentences.
**Fix**: how we resolved it.
**See**: `commit-hash` or `path/to/file.cs:line`
```

### 4. docs/tasks.md (if backlog shifted)

Update `docs/tasks.md` if any tasks moved between **To do / In progress / Done** this session. Don't rewrite — just shift the relevant lines.

### 5. Archive shipped plans/specs

Walk `docs/plans/` and `docs/specs/`. For each file:
- Shipped or superseded → `git mv` to `docs/plans/archive/` (or `docs/specs/archive/`)
- Still in flight → leave

After moving, fix cross-references with `git grep` + `Edit` (replace_all).

### 6. README.md (only if status changed)

Update only if the status line / phase / public API / architectural invariant shifted. Otherwise don't touch — README is current state, not session log.

### 7. CLAUDE.md (rare — only if orientation changed)

Update only if the read-list, tone preferences, or per-project doc list changed.

### 8. docs/HANDOFF.md (always rewritten)

Rewrite, don't append. Sections (trim what doesn't apply):

- **Session ended** — date
- **Last commit** — hash + message
- **Working tree** — clean / dirty / ahead-of-origin
- **Where we are** — current-state bullets ("Phase X is Y", not "we did Z today")
- **Next session options** — 2-3 labelled `(a)/(b)/(c)` choices, each a few lines
- **Validation pending** — what's unverified, by platform if relevant
- **Key file references** — paths the session touched

### 9. Project-specific docs (per CLAUDE.md)

If `CLAUDE.md` lists project-specific docs to keep current (architecture plans, phase docs, etc.), update them per the project's conventions. The skill doesn't know what those are — read CLAUDE.md and follow its guidance.

## Sync to Notion (after repo is sorted)

Only do this if Notion MCP tools are loaded (`mcp__*notion*`) and the writes are quick. If they fail or hang, skip and continue — Notion is a reference layer, the repo is authoritative.

**Find the matching Notion project page.** Search for the repo / display name. If ambiguous, ask once.

**Mirror to Notion:**
- **Devlog** entry → new page in the project's Devlog database (Project relation set). Body = same content as `docs/devlog/<file>.md`.
- **Decisions** entries → new page per decision in the Decisions database.
- **Gotchas** entries → new page per gotcha in the Gotchas database with Number, Title, What, Fix, See, Session.

If something fails, surface it in the summary — don't retry blindly.

## Asking about decisions

If the *why* isn't clear, ask before writing. Lead with a best-guess motive — faster than a blank question. Batch questions. If the user says "just write what you've got", flag inferred motives with `— inferred; correct if wrong.`

## Commit the doc sweep

**Verify code is committed first.** If uncommitted source exists, stop and ask. Doc commits should reference hashes that exist.

**Stage only what this skill touched:**
- New `docs/devlog/<file>.md`
- New `docs/decisions/<file>.md`
- Updated `docs/gotchas.md`, `docs/tasks.md`, `docs/HANDOFF.md`
- `README.md` (if updated)
- `CLAUDE.md` (if updated)
- `git mv`'d archived plans/specs (the renames)
- Project-specific docs from §9

**Don't stage:**
- Harness scratch (`.claude/settings.local.json`, etc.)
- ProjectSettings drift, scene-meta drift, anything tangential — surface in summary so the user reviews
- Anything you didn't expect — ask before staging

**Commit message:**

```
docs: end-project sweep — <one-line summary>

Session close-out:
- New devlog entry: <filename>.
- HANDOFF rewritten.
- [other relevant bullets]

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Use a HEREDOC. **Don't push** (user handles that). **Don't amend** prior commits.

## Behavioral notes

- **Don't invent facts.** Check `git log` if uncertain.
- **Don't duplicate content.** Devlog owns session history. HANDOFF owns next-session pointer. README owns current state. Decisions own architectural rationale. Pick one home.
- **Preserve style.** Match the existing tone in each doc.
- **Repo before Notion.** If Notion sync fails, the repo is still authoritative — finish the commit and surface the Notion failure.

## Final summary

After the commit lands, show:
- Files committed (with hash)
- Files left unstaged (so user knows what's still in working tree)
- Notion sync status (synced / partial / skipped)
- Inferred motives flagged
- Push status: "not pushed; up to you"

Then stop.
