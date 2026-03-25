# Claude Code Project Structure — Best Practices

Sources: official Claude Code docs + shanraisshan/claude-code-best-practice (Boris Cherny, Thariq Shihipar)

---

## Correct .claude/ layout

```
project/
├── CLAUDE.md                     # Project overview, < 200 lines
├── .claude/
│   ├── settings.json             # Shared permissions — commit to git
│   ├── settings.local.json       # Personal overrides — gitignored
│   ├── rules/                    # Path-scoped rules, loaded on demand
│   │   ├── backend.md            # paths: ["src/api/**", "tests/**"]
│   │   ├── frontend.md           # paths: ["src/frontend/**"]
│   │   └── testing.md            # paths: ["**/*.test.*"]
│   ├── agents/                   # Subagent definitions — isolated workers
│   │   ├── ...
│   ├── commands/                 # Human-invoked slash commands (CLI only)
│   │   ├── ...
│   └── skills/                   # Structured reusable workflows
│       └── my-skill/
│           ├── SKILL.md
│           └── scripts/
├── .mcp.json                     # MCP server config (optional)
└── ... rest of project
```

---

## The five primitives — quick reference

| Primitive | Key trait | Use when |
|-----------|-----------|----------|
| **CLAUDE.md** | Always loaded, shared context | Project overview, build commands, top-level conventions |
| **Rules** | Path-scoped, loaded on demand | Domain-specific conventions that only apply to certain files |
| **Commands** | Injected into current context | Human-triggered daily workflows (`/pipeline`, `/review`) |
| **Agents** | Fresh isolated context | Any task needing isolation: review, QA, parallel implementation |
| **Skills** | Like commands + context fork + shell injection | Reusable workflows with dynamic data or supporting files |

---

## CLAUDE.md — rules for writing it

- **Hard limit: 200 lines.** Claude starts ignoring content past that.
- Target 60 lines for best reliability (humanlayer pattern).
- Include: build/test/run commands, one-paragraph architecture, non-obvious conventions.
- Exclude: domain-specific rules (→ `rules/`), step-by-step workflows (→ commands/skills).
- Use `@path/to/file` imports to reference supporting docs without bloating the main file.
- Wrap critical rules in `<important if="...">` tags — increases compliance as file grows.
- **Litmus test:** a new dev opens the project, says "run the tests" to Claude. If it works on first try — CLAUDE.md is good. If not — something's missing.

---

## Agents — rules for writing them

- **`description` is the most important field** — it's how Claude decides when to auto-spawn the agent. Write it as a trigger: "Use when X", not "Does X".
- Reviewer/QA agents: always add `disallowedTools: Write, Edit` — physical enforcement of read-only.
- Use `model: opus` for planning and architecture agents. `model: sonnet` for implementation. `model: haiku` for cheap/fast analysis.
- Keep system prompts focused on role + gotchas. Put domain conventions in `rules/` files instead.
- Add a **Gotchas section** — most valuable part of any agent prompt, lists known failure modes.

---

## Skills — rules for writing them

- Skills are **folders**, not files. Use subdirectories: `references/`, `scripts/`, `examples/`.
- Use `context: fork` when you want main context to see only the final result, not intermediate steps.
- Use `` !`command` `` syntax to inject live shell output: `` Current branch: !`git branch --show-current` ``
- `description` field is a trigger, not a summary — write for the model.
- Don't state the obvious — focus on what pushes Claude out of default behavior.
- Don't prescribe step-by-step — give goals and constraints, let Claude decide the path.
- Include a **Gotchas section** with failure modes you've encountered.

---

## Permissions — rules for settings.json

Use wildcard syntax, not broad grants:
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Edit(src/**)",
      "Bash(git commit*)"
    ],
    "defaultMode": "default"
  }
}
```

- `settings.json` — commit to git, shared team permissions
- `settings.local.json` — gitignored, personal/broad permissions for your own machine
- `disallowedTools` in agent frontmatter always overrides `settings.json` — use it for safety
- Avoid `bypassPermissions` in shared settings

---

## What to gitignore

```gitignore
.claude/settings.local.json   # Personal settings
.claude/worktrees/             # Temporary isolated worktrees
```

Everything else in `.claude/` is a team asset — commit it.

---

## Hooks — when to use them

Hooks run outside the agentic loop — they fire regardless of what Claude decides.
Use them for things that must always happen:

```json
"hooks": {
  "PostToolUse": [{"matcher": "Edit", "hooks": [{"type": "command", "command": "npm run format"}]}],
  "Stop": [{"hooks": [{"type": "command", "command": "./scripts/check-complete.sh"}]}]
}
```

Best uses: auto-format after edits, block destructive commands, verify work at session end.

---

## How path-scoping actually works

**There is no explicit link between an agent and a rules file.** The connection is implicit, triggered by file reads.

### Two types of rules

```yaml
# Type 1 — no frontmatter: loads unconditionally, every session
# .claude/rules/always-apply.md
Never commit secrets. Always write tests.
```

```yaml
# Type 2 — with paths: loads on demand, only when matching file is read
---
paths:
  - "api/**"
  - "Api.Tests/**"
---
# .claude/rules/backend.md
No EF migrations. JSONB mutations need explicit tracking.
```

### The trigger: reading a file

Path-scoped rules load when Claude **reads a file** matching the pattern — not at session start,
not on every tool use. The Read tool is what triggers loading.

```
User: "add a field to the User model"
  └─► Claude reads api/Models/User.cs          ← Read tool fires
        └─► paths: ["api/**"] matches           ← rules/backend.md loads into context
              └─► Claude now knows: no migrations, JSONB rules, error format in Russian
```

Without reading a matching file, the rule never loads.

### How agents and rules interact

An agent defined in `.claude/agents/backend-dev.md` runs in its own isolated context.
Whether it also receives path-scoped rules depends on whether it reads matching files:

```
pipeline command spawns backend-dev agent
  └─► backend-dev reads api/Controllers/AuthController.cs
        └─► [likely] rules/backend.md loads into the agent's context
              └─► agent sees project conventions
```

**Important caveat:** the official documentation does not explicitly state whether subagents
inherit path-scoped rules. The behavior is likely the same as the main session (rules load
on Read), but this is not documented and could change.

### Safe pattern: don't rely on rules alone for agents

Because rules-in-subagents is undocumented, put critical project context in **both places**:

```
rules/backend.md           → main session context (convenient)
agents/backend-dev.md      → subagent system prompt (guaranteed)
```

Rules = DRY convenience for the main session.
Agent system prompt = explicit guarantee regardless of rules loading behavior.

For non-critical conventions (style preferences, naming), rules alone is fine.
For behavior-critical rules (no migrations, JSONB tracking, auth patterns), duplicate them
in the agent's system prompt.

---

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Agent definitions in `.claude/commands/` | Move to `.claude/agents/` — commands are for human-invoked workflows only |
| Vague agent `description` | Write as trigger condition: "Use when X", not "Does X" |
| Reviewer agent can write files | Add `disallowedTools: Write, Edit, Bash` |
| CLAUDE.md over 200 lines | Move domain rules to `.claude/rules/` with path scoping |
| All context in CLAUDE.md | Rules exist for this — use them |
| `bypassPermissions` in shared settings | Use `settings.local.json` for personal broad permissions |
| Commands used as pipelines | Commands inject into current context — use agents for isolated parallel work |
