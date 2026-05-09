---
name: initialize-project
description: Use when starting a new project (any language/framework — Unity, web, C++, library, generic) that needs the standard MeatSoft setup. Builds the docs/ scaffolding silently, then interactively generates a tailored CLAUDE.md baking in the development flow, approval gates, code principles, and pushback expectations. Single entry point — replaces the older create-claude-md and create-documentation skills. Refuses to silently overwrite an existing CLAUDE.md with real content.
user_invocable: true
---

# Initialize Project

Sets up a project for the MeatSoft workflow. Two outputs:

1. **`docs/` scaffolding** — created silently, only what's missing.
2. **`CLAUDE.md`** — interactive, shown before saving.

Idempotent: safe to re-run. Won't clobber existing files.

## When to run

- New repo, regardless of language/framework.
- Existing repo missing CLAUDE.md or `docs/`.
- Refusing to silently overwrite a real CLAUDE.md — read first, ask.

## How to run

### 1. Pre-flight

Read `./CLAUDE.md` if it exists.

- Missing → continue.
- Stub-only (no Development flow / Approval semantics sections) → continue, mention you'll replace it.
- Real custom content → show the user the existing file and ask: **replace / abort**. Do not proceed without an answer. (Merge is not offered automatically — if they want to merge, they handle it manually after seeing the new render.)

### 2. Ask questions, sequentially

One at a time. One short sentence of context per question. Acknowledge each answer briefly before moving on. Do **not** batch.

a. **What kind of project is this?** Options: Unity game, Unity tool/package, web app, C++ project, library, generic/other. (Determines whether the Unity MCP section is included.)

b. **What's the project name?** (Used in the CLAUDE.md header.)

c. **(Unity projects only)** What's the Unity project folder name on disk? (Used so Claude can match the right Unity MCP instance — multiple Unity instances are usually open at once.)

d. **Anything else this CLAUDE.md should mention?** Project-specific rules, tools, conventions. Optional — "no" is a valid answer.

### 3. Build doc scaffolding (silent)

For each path below, create only if missing. Never overwrite. Track what was created vs skipped for the final summary, but do **not** narrate progress as you go.

Files and stubs:

- `README.md` — only if missing. Use stub below.
- `docs/HANDOFF.md` — stub below.
- `docs/devlog/README.md` — stub below.
- `docs/decisions/README.md` — stub below.
- `docs/specs/README.md` — stub below.
- `docs/specs/archive/README.md` — stub below.
- `docs/plans/README.md` — stub below.
- `docs/plans/archive/README.md` — stub below.
- `docs/documentation/README.md` — stub below.
- `docs/tasks.md` — stub below.
- `docs/FAQ.md` — stub below.
- `docs/gotchas.md` — stub below.

Do **not** create `CLAUDE.md` here — that's step 4.

### 4. Render and confirm CLAUDE.md

1. Fill the CLAUDE.md template using the answers from step 2. Substitute `{{PROJECT_NAME}}` and (Unity only) `{{UNITY_PROJECT_FOLDER}}`. Omit the entire **Unity MCP** section for non-Unity projects.
2. Append the answer to question (d) under **Project-specific notes** (or leave `_None._` if "no").
3. Show the full rendered file in a fenced markdown block.
4. Ask: **"Save as ./CLAUDE.md, edit, or abort?"**
   - **save** → write `./CLAUDE.md`.
   - **edit** → ask what to change, regenerate, show again, re-ask.
   - **abort** → stop. CLAUDE.md not written. (Doc scaffolding from step 3 stays.)
5. Do **not** commit.

### 5. Final summary

Report:

- Doc scaffolding: created vs already existed (group by path).
- CLAUDE.md: written / aborted, project type, Unity MCP section in or out, extras appended y/n.
- Reminder: nothing was committed; user reviews and commits.

Then stop.

## CLAUDE.md template

Use verbatim. Do not paraphrase the rules — wording matters.

````markdown
# CLAUDE.md — {{PROJECT_NAME}}

Orientation for working on this project.

## Session start

Wait for `/claude-skills:resume-project`. Do not auto-load context.

## Doc layout

- `docs/specs/` — what we're building (one file per spec).
- `docs/plans/` — how we're building it (one file per plan).
- `docs/decisions/` — architectural decisions (ADR-style).
- `docs/devlog/` — one entry per session, written by `/claude-skills:end-project`.
- `docs/HANDOFF.md` — live next-session resume point, read by `/claude-skills:resume-project`.
- `docs/tasks.md` — backlog.
- `docs/gotchas.md` — post-hoc "bit us" notes.
- `docs/FAQ.md` — project Q&A.
- `docs/documentation/` — user-facing guides + code-derived references.

## Development flow

Every feature follows this flow. Do not skip steps.

1. **Branch off main.** Ask the user for a branch name; default style is `feature/<short-name>`.
2. **Write a spec** in `docs/specs/`. Describe *what* and *why*, not *how*.
3. **Wait for explicit spec approval** before continuing.
4. **Write a plan** in `docs/plans/`. Structure as phases × steps (e.g. 3 phases, 8 steps each). One step = one logical, committable change.
5. **Wait for explicit plan approval** before continuing.
6. **Commit the spec and plan together** once both are approved.
7. **Execute the plan one step at a time.** After each step: run tests, commit, pause. The user can object; if they don't, proceed (implicit approval).
8. **Never push** without explicit say-so.
9. **Merge to main only when the user says.** No pull requests — direct merge when instructed.

## When the plan is wrong

If executing a step reveals the spec or plan was wrong: stop, explain, propose a new path, update the spec/plan, then resume.

## Approval semantics

- **Spec and plan approval: explicit.** Wait for "approved" / "yes" / "go".
- **Per-step approval during execution: implicit.** Proceed unless the user objects.
- **Commits during plan execution: pre-authorized.** Commit after each step without re-asking.
- **Push and merge: explicit, every time.** Prior approvals do not carry forward.

## Decisions and blockers

- Do not make architectural decisions on the fly. Stop and ask.
- Significant blocker → stop and ask. Don't paper over it.
- Time, dev cost, and effort are irrelevant. Correctness beats speed.
- Small fixes mid-step (e.g. obvious failing test): attempt the fix.
- Large architectural change to fix something? Stop and ask first.
- Architectural decisions get a corresponding entry in `docs/decisions/` once made.

## Pushback is welcome

If you disagree with my plan, reasoning, or instruction — say so. I am genuinely open to being wrong, and a quiet "yes" when you see a flaw is worse than a direct objection.

- Be specific: what's flawed, why, what's better.
- Push once. If I hear you out and stick with my call, we move on.
- Don't agree just to be agreeable. I'd rather argue for two minutes than fix a bad decision for two hours.

## Code principles

- **Data oriented.** Anything expressible as pure logic should be — keep it out of framework code. This is about testability: framework-coupled code is hard to unit-test, pure logic is not. *Not* about ScriptableObjects.
- **Unit tests for everything possible. Extremely important.** Production code first, then tests.
- **In Unity:** EditMode tests by default. PlayMode tests only when the user confirms they're needed — ask first.

## Unity MCP

- Use Unity MCP whenever working with Unity. Don't fall back to manual file edits if MCP can do it.
- Multiple Unity Editor instances are usually running. Auto-detect by matching the Unity project folder name (`{{UNITY_PROJECT_FOLDER}}`) against running instances; if exactly one matches, call `set_active_instance` for it.
- If matching is ambiguous or no match is found, ask before proceeding.
- Re-confirm the active instance early in each working session.

## Tone

- Short and succinct unless asked for detail.
- Do not over-explain.
- Questions: one at a time, sequential, with a brief sentence of context. Do not batch.

## Session end

Wait for `/claude-skills:end-project`. Do not auto-wrap.

## Project-specific notes

_None._
````

## Doc scaffolding stubs

### `README.md` (only if missing)

```markdown
# <Project Name>

<one-line description>

## Status

_TODO: current status_

## Quick start

_TODO: setup instructions_
```

### `docs/HANDOFF.md`

```markdown
# Handoff

_No session has ended yet. Run `/claude-skills:end-project` after a session to populate this file._
```

### `docs/devlog/README.md`

```markdown
# Devlog

One markdown file per session, named `YYYY-MM-DD-session.md` or `session-N.md`. Append-only — past sessions stay as-is.
```

### `docs/decisions/README.md`

```markdown
# Decisions

One markdown file per architectural decision (ADR-style). Title = the decision, body = motive + alternatives considered.
```

### `docs/specs/README.md`

```markdown
# Specs

What we're building. One markdown file per spec. Move to `archive/` when shipped or superseded.
```

### `docs/specs/archive/README.md`

```markdown
# Archive

Specs that are shipped or superseded.
```

### `docs/plans/README.md`

```markdown
# Plans

How we're building it. One markdown file per plan. Move to `archive/` when shipped or superseded.
```

### `docs/plans/archive/README.md`

```markdown
# Archive

Plans that are shipped or superseded.
```

### `docs/documentation/README.md`

```markdown
# Documentation

User-facing guides and code-derived references. Organise by topic.
```

### `docs/tasks.md`

```markdown
# Tasks

Project backlog.

## To do
- [ ] _add tasks here_

## In progress

## Done
```

### `docs/FAQ.md`

```markdown
# FAQ

_No questions yet._
```

### `docs/gotchas.md`

```markdown
# Gotchas

Numbered list of post-hoc "we got bitten by X" notes.

Format: short title, what bit us, the fix, and a `See:` pointer to the commit or file.
```
