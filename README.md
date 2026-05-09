# ClaudeSkills

Personal Claude Code skills plugin for Matt Smith. Contains reusable skills available across all projects.

## Skills

| Skill | Invocation | Description |
|-------|-----------|-------------|
| create-unity-project | `/claude-skills:create-unity-project` | Create a new Unity project with GitHub repo, template, Obsidian page, and package scaffolding |
| resume-project | `/claude-skills:resume-project` | Orient at session start by reading repo docs, git state, and Obsidian project folder |
| unity-mcp-skill | `/claude-skills:unity-mcp-skill` | Orchestrate Unity Editor via MCP tools |
| hello | `/claude-skills:hello` | Test skill to verify the plugin is loaded |

## Installation (new machine)

Claude Code's `/plugin` command is not available in all environments. Install by editing `~/.claude/settings.json` directly.

Add the marketplace to `extraKnownMarketplaces` and enable the plugin in `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "claude-skills@claude-skills": true
  },
  "extraKnownMarketplaces": {
    "claude-skills": {
      "source": {
        "source": "github",
        "repo": "meatsoftgames/ClaudeSkills"
      }
    }
  }
}
```

The marketplace key (`claude-skills`) must match the `name` field in `.claude-plugin/marketplace.json`. The plugin identifier is `<plugin-name>@<marketplace-name>` — both are `claude-skills` here.

Then restart Claude. Verify with `/claude-skills:hello` — should respond "claude-skills plugin is working!".

## Local path

Skills live at: `E:\Personal\ClaudeSkills`

## Adding a new skill

1. Create `claude-skills/skills/<skill-name>/SKILL.md`
2. Add frontmatter: `name`, `description`, `user_invocable: true`
3. Push to GitHub
4. Restart Claude

## Structure

```
ClaudeSkills/
├── .claude-plugin/
│   └── marketplace.json          ← marketplace catalog
└── claude-skills/
    ├── .claude-plugin/
    │   └── plugin.json           ← plugin manifest
    └── skills/
        ├── create-unity-project/
        │   └── SKILL.md
        ├── resume-project/
        │   └── SKILL.md
        ├── unity-mcp-skill/
        │   ├── SKILL.md
        │   └── references/
        │       ├── tools-reference.md
        │       └── workflows.md
        └── hello/
            └── SKILL.md
```
