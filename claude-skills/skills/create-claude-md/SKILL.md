---
name: create-claude-md
description: Use when starting a new project and a CLAUDE.md is needed that bakes in the standard MeatSoft development flow, code rules, and approval gates. Asks about project type and name, generates a tailored CLAUDE.md, shows it for review, then saves on confirmation. Should be run early in a new project — typically before /claude-skills:create-documentation. Also use to replace a thin or stub CLAUDE.md with the full rules version. Refuses to overwrite an existing non-stub CLAUDE.md without explicit confirmation.
user_invocable: true
---

# Create CLAUDE.md

Generates the project's CLAUDE.md with all standard processes, rules, and approval gates baked in. Interactive — asks for project context, shows the rendered file, then writes it on confirmation.

## When to run

- New repo, before or alongside `/claude-skills:create-documentation`.
- Existing repo where CLAUDE.md is missing, a thin stub, or out of date.
- Do NOT silently overwrite a CLAUDE.md that has real custom content. Read it first; if it's non-trivial, show it to the user and ask whether to replace, merge, or abort.

## How to run

Ask the questions below **one at a time, sequentially**. After each answer, give a one-line acknowledgement and move on. Do not batch the questions. Keep each question to one short sentence of context.

### Pre-flight

1. Check whether `./CLAUDE.md` already exists.
   - If missing → continue.
   - If it exists and is just the `create-documentation` stub (no rules, no flow) → continue, mention you'll replace it.
   - If it exists with real content → show the user the existing file and ask: replace / merge new sections in / abort. Do not proceed without an answer.

### Questions

Ask in order:

1. **What kind of project is this?** Options: Unity game, Unity tool/package, library, web app, generic/other. (Determines whether the Unity MCP section is included.)
2. **What's the project name?** (Used in the CLAUDE.md header.)
3. **(Unity projects only)** What's the Unity project folder name on disk? (Used so Claude can match the right Unity MCP instance — there are usually multiple Unity instances open at once.)
4. **Anything else this CLAUDE.md should mention?** Project-specific rules, tools, conventions. Optional — "no" is a valid answer.

### Render and confirm

1. Fill the template below using the answers. Substitute `{{PROJECT_NAME}}` and (Unity only) `{{UNITY_PROJECT_FOLDER}}`. Omit the entire **Unity MCP** section for non-Unity projects.
2. Append the answer to question 4 under **Project-specific notes** (or leave `_None._` if they said no).
3. Show the full rendered file to the user in a fenced markdown block.
4. Ask: **"Save as ./CLAUDE.md, edit, or abort?"**
   - **save** → write `./CLAUDE.md`.
   - **edit** → ask what to change, regenerate, show again, re-ask.
   - **abort** → stop. No file written.
5. Do NOT commit. The user reviews and commits themselves (or `/claude-skills:end-project` will sweep it up later).

## Template

Use this verbatim. Do not paraphrase the rules — the wording matters.

````markdown
# CLAUDE.md — {{PROJECT_NAME}}

Orientation for working on this project.

## Session start

Wait for the user to invoke `/claude-skills:resume-project`. Do not auto-load context — they will ask when ready.

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

Wait for the user to invoke `/claude-skills:end-project`. Do not auto-wrap.

## Project-specific notes

_None._
````

## Final summary

After saving, report:

- File path written (`./CLAUDE.md`).
- Project type.
- Whether the Unity MCP section was included or omitted.
- Whether project-specific notes were appended.
- Reminder: run `/claude-skills:create-documentation` next if the rest of the doc scaffolding is missing.

Then stop.
