---
name: install-auto-unity-remote-session
description: Install the ClaudeAutoStart editor script into a Unity project so opening the Editor automatically spawns a Windows Terminal tab running `claude --rc`, titled with the project folder name. Use when the user asks to add Claude auto-start, install the auto-remote-session script, or wire up a Unity project to launch Claude Code on Editor open.
user_invocable: true
---

# Install Auto Unity Remote Session

Installs `ClaudeAutoStart.cs` under `Assets/Claude/Scripts/Editor/` in the current Unity project. On Editor startup, the script spawns a Windows Terminal tab running `claude --rc` from the project root, titled `Claude: <project-folder-name>`. A per-session sentinel file in `%TEMP%` prevents respawning on every recompile.

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
    static ClaudeAutoStart()
    {
        // Sentinel file so we only spawn once per Unity session, not on every recompile
        var sentinel = Path.Combine(Path.GetTempPath(),
            $"claude_rc_{Application.dataPath.GetHashCode():X}.lock");

        if (File.Exists(sentinel)) return;
        File.WriteAllText(sentinel, System.DateTime.Now.ToString());

        // Clean up sentinel when editor quits
        EditorApplication.quitting += () => {
            if (File.Exists(sentinel)) File.Delete(sentinel);
        };

        var projectRoot = Path.GetDirectoryName(Application.dataPath);
        var psi = new ProcessStartInfo
        {
            FileName = "wt.exe",
            Arguments = $"new-tab --title \"Claude: {Path.GetFileName(projectRoot)}\" " +
                        $"--startingDirectory \"{projectRoot}\" cmd /k claude --rc",
            UseShellExecute = true,
            CreateNoWindow = false,
        };
        Process.Start(psi);
        UnityEngine.Debug.Log($"[ClaudeAutoStart] Spawned Remote Control session for {projectRoot}");
    }
}
```

Do not generate a `.meta` file — Unity creates it on next refresh.

## Step 5: Report

Tell the user:
- File installed at `Assets/Claude/Scripts/Editor/ClaudeAutoStart.cs`.
- On the next time the Unity Editor opens (or after a recompile in a fresh session), a Windows Terminal tab will spawn titled `Claude: <project-folder-name>` running `claude --rc` from the project root.
- The sentinel lock file at `%TEMP%\claude_rc_<hash>.lock` prevents duplicate spawns within a single Editor session and is cleaned up on quit.
- Requirements: `wt.exe` (Windows Terminal) and `claude` must be on PATH.

## Notes

- Editor scripts under any folder named `Editor/` are auto-treated as Editor-only by Unity, so no `.asmdef` is required.
- If the Unity project uses a global `.asmdef` that excludes new folders, the user may need to add a reference; flag this only if compilation errors occur.
