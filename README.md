# claude-code-kit

Reusable agents, skills, commands, and rules for Claude Code — applicable to any project, any tech stack.

## Installation

```bash
claude plugins install github:<your-username>/claude-code-kit
```

Or for local development:

```bash
claude plugins install /path/to/claude-code-kit
```

## What's included

### Agents

| Agent | Trigger | Model |
|---|---|---|
| `planner` | Before writing any code on a non-trivial task | opus |
| `code-reviewer` | After code changes are complete | opus |
| `security-reviewer` | When touching auth, input handling, or sensitive data | opus |
| `architect` | When the right structural approach is unclear | opus |
| `implementer` | When a plan exists and implementation starts | sonnet |
| `debugger` | When a bug exists and root cause is unclear | sonnet |
| `verifier` | After implementation — runs tests and build | sonnet |

### Commands

| Command | What it does |
|---|---|
| `/pipeline <task>` | Full flow: plan → implement → review → verify |
| `/pipeline-review [files]` | Review only, no code changes |
| `/pipeline-bugfix <description>` | Fast: diagnose → fix → verify |

### Rules

`rules/universal.md` — always loaded, applies to every session:
- No secrets in files
- New code requires tests
- Reviewer agents never modify files
- Follow existing patterns over optimization
- Parallel agents require explicit phrase

## How overrides work

Plugin primitives load first. Project-level files with the same name override them:

```
Plugin agents/code-reviewer.md
  └─► Project .claude/agents/code-reviewer.md  ← wins if exists
```

To customize an agent for your project, create `.claude/agents/<name>.md` — it replaces the plugin version entirely for that project.

## Reference documentation

See [docs/](docs/) for methodology guides:
- [Orchestration setup guide](docs/orchestration-setup-guide.md)
- [Project structure best practices](docs/project-structure-best-practices.md)
- [Settings and permissions](docs/settings-and-permissions.md)
- [Multiple pipelines pattern](docs/multiple-pipelines.md)
- [Creating your own plugin](docs/creating-your-own-plugin.md)
