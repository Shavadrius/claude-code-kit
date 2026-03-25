# Claude Code — Settings & Permissions

## Settings hierarchy (most specific wins)

```
~/.claude/settings.json          ← global, all projects
  └─► .claude/settings.json      ← project, shared (commit to git)
        └─► .claude/settings.local.json  ← personal, gitignored
```

Changes in a more specific file override the parent — they don't merge arrays,
the entire `permissions.allow` array is replaced at each level.

---

## What to put where

| Setting | Where |
|---------|-------|
| Permissions your whole team needs | `.claude/settings.json` (commit) |
| Personal broad permissions on your machine | `.claude/settings.local.json` (gitignored) |
| Global defaults across all your projects | `~/.claude/settings.json` |
| Model, plugins, effortLevel | `~/.claude/settings.json` (personal preference) |

---

## Permissions analysis for this project

### Already in `.claude/settings.json` (shared)
```json
"Bash(dotnet*)"        ← build, test, run
"Bash(npm*)"           ← frontend build/test/install
"Bash(docker compose*)" ← stack management
```

### Missing — should be added to `.claude/settings.json`
```json
"Bash(git status)",
"Bash(git log*)",
"Bash(git diff*)",
"Bash(git add*)",
"Bash(git commit*)"
```
Git commands are needed for every commit workflow. Without them Claude asks permission on every `git status`.

### In `settings.local.json` — review periodically
The local file accumulates one-off permissions from past sessions:
- `Bash(find ...)` — was needed once for exploration, not a project staple
- `Bash(head -30 ...)` — one-off inspection command
- `Bash(xargs -I {} basename {})` — one-off exploration

**Rule:** if a command appears in `settings.local.json` and you've used it more than once,
promote it to `settings.json`. Otherwise clear it periodically.

### In `~/.claude/settings.json` (global)
```json
"Bash(docker exec:*)"     ← running commands inside containers
"Bash(curl:*)"            ← HTTP requests from shell
"Bash(git:*)"             ← already broad at global level
"WebSearch"               ← web search without confirmation
"defaultMode": "dontAsk"  ← all allowed commands skip confirmation
```

`defaultMode: dontAsk` globally means Claude never asks for non-listed commands if they're not
explicitly denied. Convenient but broad — acceptable for a personal dev machine.

---

## Permission syntax reference

```json
"Bash(npm run *)"           // wildcard — any npm run subcommand
"Bash(git commit*)"         // prefix match — git commit, git commit -m, etc.
"Edit(src/**)"              // path-scoped edit permission
"Read(//root/projects/**)"  // absolute path read
"WebFetch(domain:github.com)" // specific domain only
"WebSearch"                 // entire tool
"Agent(code-reviewer)"      // specific agent invocation
```

**Wildcards:**
- `*` matches anything in one path segment
- `**` matches across path separators
- No wildcard = exact match

---

## Recommended `.claude/settings.json` for this project

```json
{
  "permissions": {
    "allow": [
      "Bash(dotnet*)",
      "Bash(npm*)",
      "Bash(docker compose*)",
      "Bash(git status)",
      "Bash(git log*)",
      "Bash(git diff*)",
      "Bash(git add*)",
      "Bash(git commit*)"
    ]
  }
}
```

Keep `defaultMode` out of project settings — it's a personal preference, belongs in `~/.claude/settings.json`.

---

## Useful global settings (`~/.claude/settings.json`)

```json
{
  "model": "opus",              // default model for main session
  "effortLevel": "medium",      // thinking budget: low / medium / high / max
  "permissions": {
    "defaultMode": "dontAsk",   // skip confirmation for allowed commands
    "allow": [...]
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

**effortLevel** controls how much Claude thinks before responding:
- `low` — fast, cheap, simple tasks
- `medium` — default, balanced
- `high` / `max` — complex architecture, debugging, planning

---

## Maintenance habit

Review `settings.local.json` monthly:
- Promote recurring commands to `settings.json`
- Delete one-off commands that accumulated from exploration sessions
- If it's empty — delete the file entirely
