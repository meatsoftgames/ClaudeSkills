---
name: install-auto-unity-remote-session
description: Install the ClaudeAutoStart editor script into a Unity project so opening the Editor automatically spawns a Windows Terminal tab running `claude --rc`, titled with the project folder name. Use when the user asks to add Claude auto-start, install the auto-remote-session script, or wire up a Unity project to launch Claude Code on Editor open.
user_invocable: true
---

# Install Auto Unity Remote Session

Installs `ClaudeAutoStart.cs` under `Assets/Claude/Scripts/Editor/` in the current Unity project. On Editor startup, the script spawns a Windows Terminal tab running `claude --rc` from the project root, titled `Claude: <project-folder-name>`. A per-session sentinel file in `%TEMP%` prevents respawning on every recompile. A `MeatSoft/AI/Start Claude Remote Session` menu item lets the user relaunch the session manually (e.g. after closing the tab) without reopening the project.

## Step 1: Verify Unity project

The current working directory must be a Unity project root. Check that **both** of these exist:
- `Assets/`
- `ProjectSettings/`

If either is missing, stop and tell the user:
> Not in a Unity project root. Run this from the folder containing `Assets/` and `ProjectSettings/`.

## Step 2: Ensure target folder

Make sure the directory `Assets/Claude/Scripts/Editor/` exists. Create any missing parents.

## Step 3: Existence check

If `Assets/Claude/Scripts/Editor/ClaudeAutoStart.cs` already exists:
1. Tell the user the file is already installed.
2. Ask: **overwrite or skip?**
3. If skip → stop, report no change.
4. If overwrite → continue.

## Step 4: Write the script

Write the following content **verbatim** to `Assets/Claude/Scripts/Editor/ClaudeAutoStart.cs`:

```csharp
using UnityEditor;
using UnityEngine;
using System.Diagnostics;
using System.IO;

[InitializeOnLoad]
public static class ClaudeAutoStart
{
    const string SessionKey = "ClaudeAutoStart.Spawned";

    static ClaudeAutoStart()
    {
        if (SessionState.GetBool(SessionKey, false)) return;
        SessionState.SetBool(SessionKey, true);
        SpawnSession();
    }

    [MenuItem("MeatSoft/AI/Start Claude Remote Session")]
    public static void SpawnSession()
    {
        var projectRoot = Path.GetDirectoryName(Application.dataPath);
        var projectName = Path.GetFileName(projectRoot);
        var sessionName = $"{projectName}-remote";
        var title = $"Claude: {projectName}";
        var psi = new ProcessStartInfo
        {
            FileName = "wt.exe",
            Arguments = $"new-tab --suppressApplicationTitle --title \"{title}\" " +
                        $"--startingDirectory \"{projectRoot}\" cmd /k claude --remote-control \"{sessionName}\"",
            UseShellExecute = true,
            CreateNoWindow = false,
        };
        Process.Start(psi);
        UnityEngine.Debug.Log($"[ClaudeAutoStart] Spawned Remote Control session '{sessionName}' for {projectRoot}");
    }
}
```

Do not generate a `.meta` file — Unity creates it on next refresh.

## Step 5: Report

Tell the user:
- File installed at `Assets/Claude/Scripts/Editor/ClaudeAutoStart.cs`.
- On the next time the Unity Editor opens (or after a recompile in a fresh session), a Windows Terminal tab will spawn titled `Claude: <project-folder-name>` running `claude --remote-control "<project-folder-name>-remote"` from the project root, so the remote-control session is registered as `<project-folder-name>-remote` (e.g. `reaver-remote`).
- The `MeatSoft > AI > Start Claude Remote Session` menu item relaunches the session manually whenever needed.
- Unity's `SessionState` prevents duplicate spawns within a single Editor session and is automatically cleared when the Editor exits (even on crash/force-close), so the next launch always spawns cleanly.
- Requirements: `wt.exe` (Windows Terminal) and `claude` must be on PATH.

## Notes

- Editor scripts under any folder named `Editor/` are auto-treated as Editor-only by Unity, so no `.asmdef` is required.
- If the Unity project uses a global `.asmdef` that excludes new folders, the user may need to add a reference; flag this only if compilation errors occur.
