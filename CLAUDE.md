# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A knowledge base of Claude Code orchestration patterns and best practices — not a runnable application. No build system, no tests, no dependencies.

The content is technology-agnostic methodology drawn from official Claude Code docs and community sources (shanraisshan/claude-code-best-practice, Boris Cherny, Thariq Shihipar).

## Repository structure

All files are markdown documentation at the root level:

- `orchestration-setup-guide.md` — The main guide: 10-step methodology for setting up agent orchestration from scratch. Covers all five Claude Code primitives (CLAUDE.md, rules, commands, agents, skills), pipeline design, permissions, hooks, and testing.
- `project-structure-best-practices.md` — Reference card for `.claude/` directory layout, rules for writing each primitive, path-scoping mechanics, and common mistakes.
- `creating-your-own-plugin.md` — Two approaches to reusable Claude Code configs: personal plugins (installed globally via `claude plugins install`) and project template repos. Covers plugin structure, marketplace registration, layered loading model.
- `multiple-pipelines.md` — Pattern for multiple pipeline commands (feature, bugfix, review, refactor, hotfix) with agent allocation per pipeline type.
- `settings-and-permissions.md` — Settings hierarchy (`global → project → local`), permission wildcard syntax, and recommendations for what goes where.
- `commands-cli-vs-vscode.md` — Finding: custom `/commands` only work in terminal CLI, not VS Code extension. Workarounds documented.
- `helpers/` — Empty directory, placeholder for future helper scripts.

## Key concepts across the docs

- **Five primitives**: CLAUDE.md (always loaded), Rules (path-scoped), Commands (human-triggered), Agents (isolated context), Skills (commands + fork + shell injection)
- **Agent description field**: Write as trigger condition ("Use when X"), not job title — this is how Claude decides auto-delegation
- **Path-scoped rules**: Load when Claude reads a matching file, not at session start. Undocumented whether subagents inherit them — duplicate critical rules in agent system prompts
- **Parallel execution**: Requires explicit phrasing "spawn X and Y IN PARALLEL (single message, multiple Agent tool calls)"
- **CLAUDE.md size**: Hard limit ~200 lines, target ~60 for best reliability

## When editing these docs

- Maintain technology-agnostic tone — examples use .NET/Vue but the patterns are universal
- Each file is self-contained; cross-references are by concept, not by link
- The orchestration setup guide follows a numbered step sequence (Steps 1-10) — preserve this structure
