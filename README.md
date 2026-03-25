# claude-code-kit

Reusable agents, skills, commands, and rules for Claude Code — applicable to any project, any tech stack.

## Installation

This is a **private plugin** — `claude plugins install github:...` works only for plugins in the official Anthropic marketplace. Use manual installation instead:

```bash
# Clone to the Claude plugins directory
git clone git@github.com:Shavadrius/claude-code-kit.git \
  ~/.claude/plugins/marketplaces/claude-code-kit

# Enable the plugin
claude plugins enable claude-code-kit
```

If using SSH with a custom config alias (e.g. `github-claude-code-kit`):
```bash
git clone git@github-claude-code-kit:Shavadrius/claude-code-kit.git \
  ~/.claude/plugins/marketplaces/claude-code-kit

claude plugins enable claude-code-kit
```

For local development (from a local path):
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
| `researcher` | When asked to research or investigate any topic | sonnet |

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

## Configuration

### Research output path

By default, the `researcher` agent saves files to `research/` in your current project.

To override per-project, add this line to your project's `CLAUDE.md`:

```
Research output path: docs/research/
```

The agent reads `CLAUDE.md` on each invocation and uses the configured path.

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
