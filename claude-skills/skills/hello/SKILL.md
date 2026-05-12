---
name: hello
description: Test skill to verify the claude-skills plugin is loaded correctly. Reads the plugin version from plugin.json and echoes it back so the user can confirm the installed version matches what's in the repo.
user_invocable: true
---

# Hello

Read the `version` field from `claude-skills/.claude-plugin/plugin.json` (relative to the plugin root — use the path your tools actually see).

Then reply with exactly one line:

> claude-skills v<VERSION> is working!

Substitute `<VERSION>` with the value you just read. No preamble, no trailing summary.

If you can't read the manifest for some reason, reply:

> claude-skills is working (version unknown — could not read plugin.json)
