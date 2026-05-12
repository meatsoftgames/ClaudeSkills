# Changelog

Versioned log of changes to the `claude-skills` plugin. To validate the plugin is current on any machine, run `/claude-skills:hello` — it echoes the version that actually loaded.

## v0.2.0 — 2026-05-12

- `end-project`: added a **Cold-start check** step — read the docs you just wrote as a fresh agent with zero context, fix gaps before committing.
- `end-project`: **Notion sync is now non-negotiable** — stop if MCP tools missing, diagnose/retry on write failure, no silent skip.
- `end-project`: new §9 — **`docs/plugin-reference.md` is updated without fail** when the session touches plugin shape/behavior (skipped only if the file doesn't exist).
- `resume-project`: read-order rewritten to **CLAUDE.md → PROJECT.md → docs/HANDOFF.md → docs/plugin-reference.md (if present) → git state → active branch spec/plan**. Obsidian section removed.
- `initialize-project`: new question asks whether the project is a plugin (Unity package, Claude plugin, VSCode extension, etc.). If yes, scaffolds `docs/plugin-reference.md` from a stub and keeps the matching line in the CLAUDE.md doc-layout section.
- `hello`: now echoes the plugin version for runtime validation.

## v0.1.0 — initial

- Initial set of skills: `hello`, `initialize-project`, `resume-project`, `end-project`, `create-unity-project`, `install-auto-unity-remote-session`, `unity-mcp-skill`.
