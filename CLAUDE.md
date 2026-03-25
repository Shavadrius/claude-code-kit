# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**claude-code-kit** — an installable Claude Code plugin providing reusable agents, skills, commands, and rules applicable to any project and tech stack.

Install via: `claude plugins install github:<user>/claude-code-kit`

## Repository structure

### Executable primitives (installed as plugin)

- `agents/` — 7 reusable agents: planner, code-reviewer, security-reviewer, architect, implementer, debugger, verifier
- `commands/` — 3 pipeline commands: pipeline, pipeline-review, pipeline-bugfix
- `rules/universal.md` — always-on rules loaded in every session
- `skills/` — empty, Phase 6 (tdd-workflow, git-commit, explore-codebase, api-design, estimate-complexity)
- `hooks/` — empty, examples in docs

### Plugin metadata

- `.claude-plugin/plugin.json` — plugin identity (name, version, description)
- `.claude-plugin/marketplace.json` — marketplace registration schema
- `package.json` — npm metadata with files list

### Reference documentation

- `docs/orchestration-setup-guide.md` — 10-step methodology for agent orchestration from scratch
- `docs/project-structure-best-practices.md` — `.claude/` directory layout, rules for writing each primitive
- `docs/creating-your-own-plugin.md` — plugin structure, marketplace registration, layered loading model
- `docs/multiple-pipelines.md` — multiple pipeline pattern with agent allocation per type
- `docs/settings-and-permissions.md` — settings hierarchy, permission wildcard syntax
- `docs/commands-cli-vs-vscode.md` — custom `/commands` only work in CLI, not VS Code extension

## Key concepts

- **Agent description field**: Write as trigger condition ("Use when X"), not job title
- **Reviewer agents**: Always use `disallowedTools: Write, Edit, Bash` — enforced in frontmatter
- **Parallel execution**: Requires phrase "IN PARALLEL (single message, multiple Agent tool calls)"
- **Plugin loading order**: Plugin → user-level `~/.claude/` → project-level `.claude/` (last wins)
- **Override pattern**: Create `.claude/agents/<name>.md` in a project to replace the plugin version

## When editing this plugin

- Keep agent system prompts ≤ 50 lines — longer prompts dilute focus
- Every agent needs at least 2 Gotchas (real failure modes, not obvious advice)
- `description` field must be a trigger condition, not a role name
- Technology-agnostic — no stack-specific assumptions in any primitive
- Skills go in `skills/<name>/SKILL.md` (directory, not file)
