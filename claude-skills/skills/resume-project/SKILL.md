---
name: resume-project
description: Use this skill when the user says "refresh yourself on the project", "get up to speed", "resume project", or an obvious variant that asks you to orient on current project state before proceeding (e.g., "catch me up", "where were we", "what's the state of things"). Reads CLAUDE.md, PROJECT.md, docs/HANDOFF.md, docs/plugin-reference.md (if present), the active branch's spec/plan, and current git state, then summarizes what was last completed plus any handoff notes. The user invokes this at session start to re-ground on the latest work and confirm you have current context — don't skip it when they ask. The goal is a tight "here's where we left off" reminder, not a full project tour.
---

# Resume Project

Orient at the start of a session by reading the project's standing docs and the active branch's work-in-progress, then summarize where the last session left off.

## Why this exists

At the start of a session — whether after a break, a crash, or just the next day — context from the previous session is gone. The user wants a quick reminder of "what were we doing, what shipped, what's next" and they want you grounded in the latest state before you suggest anything. This skill is the mechanical "go read the right files in the right order" checklist that makes that happen reliably.

The output is a short summary, not an essay. The user wrote the docs; they don't need them replayed verbatim. They need the pointer.

## What to read, in order

### 1. CLAUDE.md

Read `CLAUDE.md` at the repo root first. It carries the orientation: read-order, tone preferences, doc layout, development flow. Follow whatever it specifies.

If `CLAUDE.md` is missing, note that in the summary — the project hasn't been initialized with the standard scaffolding.

### 2. PROJECT.md

Read `PROJECT.md` at the repo root next. This is the project's specifics — architecture, project rules, best practices, conventions. If it's a stub or missing, note that and continue.

### 3. docs/HANDOFF.md

Read `docs/HANDOFF.md` — the explicit "where we left off / what's next" pointer written by the previous session's `end-project`. This is the single highest-value file for re-grounding.

### 4. docs/plugin-reference.md (if it exists)

If `docs/plugin-reference.md` is present, read it. It's a quick "how does this whole thing work" overview for plugin projects (Unity packages, Claude plugins, etc.) — saves you from reading the entire codebase to understand the shape of the thing.

If the file isn't there, skip silently — not every project is a plugin project.

### 5. Git state

Run in parallel:
- `git log --oneline -15` — what recently shipped
- `git status` — uncommitted or untracked files (often = in-progress work from last session)
- `git stash list` — stashed work, easy to miss

Note whether the branch is ahead of origin (something committed but unpushed) or if there's a dirty working tree that doesn't match HANDOFF's state — both are real signals.

### 6. Active branch context (if not on `main`/`master`)

If `git branch --show-current` returns anything other than `main` or `master`, you're mid-feature. Before continuing, read the work-in-progress docs for this branch:

1. **Find the spec.** Look in `docs/specs/` for a file whose name matches the branch's short name (strip the `feature/` prefix and any leading date). Try exact match first, then fuzzy (substring, kebab-case variants). If multiple candidates, list them and ask which is the active one.
2. **Read the spec in full.** This is the *what* and *why* of the branch — short enough to read whole.
3. **Find and read the master plan** for that spec. Same name match, but in `docs/plans/`. If the plan references sub-plans (e.g. "see `docs/plans/<name>-phase-2.md`" or links to other plan files), read those too — sub-plans first to last.
4. **Cross-check progress** against the plan's `- [ ]` / `- [x]` checkboxes vs. the commits on the branch since it diverged from main: `git log main..HEAD --oneline`. Note any drift between the plan and the actual commits.
5. If there's no spec/plan for this branch, say so explicitly — that branch is operating on the lighter flow, and the devlog/git history is the only record.

Skip this step entirely on `main`/`master` — it's not relevant.

## What to report

Short. A small paragraph or a short bulleted list. Not a document.

Include:

1. **What last shipped.** One sentence, with the commit hash or devlog date. E.g., "Last shipped: `68ac052` — the frame-driver perf fix that took 2000 instances from 0.08 FPS to 50 FPS."
2. **Working-tree state.** Anything uncommitted or unpushed? If clean, say "clean, up to date with origin."
3. **Next-session notes.** If `HANDOFF.md` exists, or the most recent devlog has an explicit next-session section, surface the items — paraphrased, not quoted in full.
4. **Readiness.** Say "ready to work" explicitly. That's the signal the user is waiting for.
5. **Clarifying questions — only if genuinely ambiguous.** If the docs cleanly point at next steps, don't invent questions. If they conflict or if there's an unresolved decision pending, ask.
6. **Active-branch state (if non-main).** Which spec/plan you read, where progress sits against the plan checkboxes, and any drift between plan and commits.

Match the user's preferred tone if CLAUDE.md or saved memory specifies it. The default is concise, direct, no preamble, no trailing summary.

## Behavioral notes

**Don't start doing the work.** Reading about pending work is not an invitation to start on it. Summarize, then wait for direction.

**Don't quote docs back verbatim.** The user wrote them. A paraphrase with the right pointer is more useful than a copy-paste.

**Don't re-read what's already in context.** If this conversation started fresh, read everything. If you're mid-session and the user just wants a re-grounding, refresh what's stale rather than duplicating what's already visible.

**Trust the reflective record over the plan.** If CLAUDE.md or a plan file says one thing but the devlog and recent commits say another, the devlog is authoritative — it's written after the fact, with actual knowledge of what happened. Flag the discrepancy as worth fixing later; don't try to reconcile it unprompted.

**Graceful degradation.** If any source is missing (no CLAUDE.md, no PROJECT.md, no HANDOFF.md, shallow git history), use what you have and note what's absent so the user knows the summary is partial.

## When things look suspicious

Some things worth mentioning in the summary even if the user didn't ask:

- **Uncommitted changes on tracked files** — often an interruption point; the user may or may not remember.
- **An untracked file that looks like in-progress work** (new `.cs`, `.md` not yet `git add`ed).
- **Branch ahead of origin** — something committed but unpushed.
- **Devlog "Next session should" items that appear not done yet** given the commit history since.
- **Open questions** flagged in docs or research notes that were marked as blockers.

Don't belabor these. Mention them as part of the summary's last line: "Heads up: there's an uncommitted change to X and a `HANDOFF.md` item about Y that looks unstarted." Then ready-to-work.
