# Custom Commands: CLI vs VS Code Extension

## Finding

Custom slash commands defined in `.claude/commands/*.md` only work in the terminal Claude Code CLI.
They do **not** appear in the VS Code extension chat.

## How to use commands

**Terminal (works):**
```bash
cd /root/projects/si-manager
claude          # start a new interactive session from the project directory
/run-tests      # commands are only visible after (re)starting the session
/review api/Controllers/AuthController.cs
```

> **Note:** If you add a new command file while a session is already running, you must exit and restart
> the session for the new command to appear. Commands are loaded at session start.

**VS Code extension (does not support custom /commands):**
- Reference the command file manually: `follow @.claude/commands/run-tests.md`
- Or just describe the task in plain language: "run all backend tests"

## Recommendation

If your team uses VS Code extension primarily — document reusable prompts as examples in `CLAUDE.md`
rather than as `/commands`. Reserve `.claude/commands/` for terminal workflow.
