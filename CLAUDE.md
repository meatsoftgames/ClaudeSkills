# CLAUDE.md — ClaudeSkills

Orientation for working on this project.

## Required tooling

The **superpowers** plugin (`obra/superpowers-marketplace`) must be installed. Its skills auto-activate — relevant ones include `writing-plans`, `executing-plans`, `test-driven-development`, `subagent-driven-development`, `requesting-code-review`. If superpowers is not available, stop and ask the user to install it before proceeding.

```
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

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

This project is a skill library. Most skills are small (often a single SKILL.md, no code) and don't need ceremony.

**Default flow:**
1. **Work on main.** No branches for typical skill edits.
2. **Brief design discussion** in chat — agree on shape before editing files. No spec doc, no plan doc.
3. **Make the changes** directly.
4. **Commit** with a clear message.
5. **Never push** without explicit say-so.

**Heavier flow (opt-in, only when the user asks):**
For substantial work — new multi-file skills, cross-cutting refactors, anything genuinely complex — use the spec → plan → execute cycle from superpowers:
1. Branch off main (`feature/<short-name>`).
2. Write a spec to `docs/specs/` together (hand-authored, no `brainstorming` skill).
3. Wait for explicit spec approval.
4. Write a plan via `writing-plans` to `docs/plans/`.
5. Wait for explicit plan approval.
6. Commit spec + plan together.
7. Execute via `executing-plans` (ask inline vs subagent-driven first).
8. Merge to main when the user says.

The user decides which flow applies. When unclear, ask.

## Worktrees

**Never use git worktrees** unless the user explicitly asks. Superpowers' `using-git-worktrees` skill is opt-in only — do not auto-activate it.

## When the plan is wrong

If executing a step reveals the spec or plan was wrong: stop, explain, propose a new path, update the spec/plan, then resume.

## Approval semantics

- **Per-edit approval: implicit.** Proceed unless the user objects.
- **Commits: pre-authorized** once the change is agreed.
- **Push and merge: explicit, every time.** Prior approvals do not carry forward.
- **Heavier flow gates** (spec approval, plan approval) only apply when the heavier flow is in use.

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
- **Test-driven.** Follow superpowers' `test-driven-development` skill — RED → GREEN → REFACTOR. Write the failing test first, then the production code that makes it pass, then refactor. Unit tests for everything possible. Extremely important.
- **In Unity:** EditMode tests by default. PlayMode tests only when the user confirms they're needed — ask first.

## Tone

- Short and succinct unless asked for detail.
- Do not over-explain.
- Questions: one at a time, sequential, with a brief sentence of context. Do not batch.

## Session end

Wait for `/claude-skills:end-project`. Do not auto-wrap.

## Project-specific notes

_None._
