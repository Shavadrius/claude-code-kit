# Creating Your Own Claude Code Plugin

## Two approaches

### Approach A — Personal plugin (recommended to start)
A git repo you install once globally (`~/.claude/`). Works across all your projects automatically.
This is what `everything-claude-code` is. You install it once, it's available everywhere.

### Approach B — Project template repo
A git repo you copy/clone as a starting point for each new project.
Contains a pre-configured `.claude/` directory. No installation mechanism — just copy and customize.

**Which to choose:**
- Agents and skills that apply to ALL your projects → **Plugin (Approach A)**
- Project-specific setup (tech stack, conventions) → **Project template (Approach B)**
- Both → Start with a template, install a plugin on top

---

## Approach A — Building a personal plugin

### Repository structure

```
my-claude-plugin/
├── .claude-plugin/
│   ├── plugin.json          # Plugin identity
│   └── marketplace.json     # Marketplace registration
├── agents/
│   ├── code-reviewer.md
│   ├── planner.md
│   └── ...
├── skills/
│   ├── my-workflow/
│   │   └── SKILL.md
│   └── ...
├── commands/
│   ├── pipeline.md
│   └── ...
├── rules/
│   └── always-follow.md     # Rules without paths: — always loaded
├── hooks/
│   └── post-edit-format.sh
├── package.json
└── README.md
```

### `.claude-plugin/plugin.json`

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My personal Claude Code configurations",
  "author": { "name": "Your Name" },
  "license": "MIT",
  "keywords": ["claude-code", "agents", "skills"]
}
```

### `.claude-plugin/marketplace.json`

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "my-plugin",
  "description": "My personal Claude Code configurations",
  "owner": { "name": "Your Name", "email": "you@example.com" },
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./",
      "version": "1.0.0",
      "category": "workflow",
      "strict": false
    }
  ]
}
```

### `package.json`

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My personal Claude Code configurations",
  "files": [
    "agents/",
    "skills/",
    "commands/",
    "rules/",
    "hooks/",
    ".claude-plugin/"
  ]
}
```

### Installing your plugin

After pushing to GitHub:
```bash
# Install from GitHub
claude plugins install github:your-username/my-plugin

# Enable it
# Claude adds this to ~/.claude/settings.json:
# "enabledPlugins": { "my-plugin@my-plugin": true }
```

Or manually (for local development):
```bash
# Clone to the plugins directory
git clone https://github.com/your-username/my-plugin \
  ~/.claude/plugins/marketplaces/my-plugin

# Register in ~/.claude/plugins/known_marketplaces.json
# Enable in ~/.claude/settings.json
```

---

## Approach B — Project template repo

Simpler. No installation mechanism. Just a repo with a pre-configured `.claude/` directory.

### Repository structure

```
my-project-template/
├── CLAUDE.md                    # Template instructions (edit per project)
├── .claude/
│   ├── settings.json            # Base permissions
│   ├── agents/
│   │   ├── code-reviewer.md     # Generic reviewer
│   │   ├── planner.md           # Generic planner
│   │   └── verifier.md          # Generic verifier
│   ├── rules/
│   │   └── general.md           # General coding rules
│   └── commands/
│       ├── pipeline.md          # Feature pipeline
│       ├── pipeline-bugfix.md   # Bugfix pipeline
│       └── review.md            # Code review
├── .gitignore                   # Includes .claude/settings.local.json
└── README.md                    # How to use this template
```

### Usage workflow

```bash
# Start a new project from template
cp -r my-project-template/. my-new-project/
cd my-new-project
git init

# Customize CLAUDE.md for the new project
# Add tech-stack-specific agents in .claude/agents/
# Add stack-specific rules in .claude/rules/
```

### Layering pattern — template + project-specific

```
my-project-template/.claude/    ← generic (from template)
  agents/code-reviewer.md       ← works for any project
  agents/planner.md             ← works for any project

my-project/.claude/             ← project-specific additions
  agents/backend-dev.md         ← .NET specific
  agents/frontend-dev.md        ← Vue specific
  rules/backend.md              ← project DB/auth conventions
  rules/frontend.md             ← project UI conventions
```

---

## Agent file format

```yaml
---
name: code-reviewer
description: Use proactively after any code changes. Reviews quality, patterns, and security.
tools: Read, Grep, Glob
disallowedTools: Write, Edit, Bash
model: sonnet
---

You are a code reviewer. [Role description]

## Review checklist
- [Item 1]
- [Item 2]

## Gotchas
- [Known failure mode 1 — this is the most valuable section]
- [Known failure mode 2]

## Output format
[How to structure the response]
```

**Rules:**
- `description` = trigger condition, not job title. Write for the model: "Use when X"
- `tools` + `disallowedTools`: reviewers get read-only, implementers get full access
- `model`: `opus` for planning/review, `sonnet` for implementation, `haiku` for cheap tasks
- Gotchas section: add failure modes you discover over time — highest-signal content

---

## Skill file format

```
skills/my-skill/
├── SKILL.md        # Required
├── template.md     # Optional output template
├── examples/       # Optional example outputs
└── scripts/        # Optional helper scripts
```

```yaml
---
name: my-skill
description: When to use this skill — written as a trigger condition for the model
context: fork      # Optional: run in isolated subagent
allowed-tools: Read, Grep
---

# My Skill

## When to Activate
- Condition 1
- Condition 2

## Instructions
[What to do]

## Gotchas
- [Known failure mode 1]

## Output Format
[How to structure the response]
```

Dynamic shell injection:
```markdown
Current status: !`git status --short`
Current branch: !`git branch --show-current`
```

---

## Scoping: what goes in a plugin vs a project

| Belongs in plugin | Belongs in project `.claude/` |
|-------------------|-------------------------------|
| Language-agnostic reviewers (code-reviewer, security-reviewer) | Tech-stack-specific agents (backend-dev with .NET context) |
| Generic pipelines (feature, bugfix, review) | Project-specific pipelines with project rules |
| Universal skills (api-design, tdd-workflow) | Domain knowledge (Tournament.Data is JSONB, etc.) |
| Always-on rules (no secrets in code, test coverage) | Stack-specific conventions (PrimeVue auto-import) |

---

## Versioning and sharing

```
my-plugin/
├── CHANGELOG.md        # What changed in each version
├── .claude-plugin/
│   └── plugin.json     # Bump "version" here on each release
└── package.json        # Same version here
```

**Semantic versioning:**
- `patch` (1.0.1) — improved agent instructions, better skill examples
- `minor` (1.1.0) — new agent or skill added
- `major` (2.0.0) — breaking change to agent interface or removed component

**For a team:** host on internal GitHub, everyone installs the same plugin version.
Pin a specific version per project in `settings.json` to avoid surprise updates.

---

## Registering a private plugin repository

Claude Code discovers plugins through `~/.claude/plugins/known_marketplaces.json`.
You can add your own private GitHub repo as a marketplace:

```bash
# Register your marketplace (public or private repo)
claude plugins add marketplace github:your-org/your-plugin-repo

# This adds an entry to known_marketplaces.json:
# {
#   "your-plugin-repo": {
#     "source": { "source": "github", "repo": "your-org/your-plugin-repo" },
#     "installLocation": "~/.claude/plugins/marketplaces/your-plugin-repo"
#   }
# }

# Then install the plugin from your marketplace
claude plugins install your-plugin@your-plugin-repo

# Enable it (or Claude adds this automatically)
# In ~/.claude/settings.json:
# "enabledPlugins": { "your-plugin@your-plugin-repo": true }
```

**Private repos:** use a GitHub token with `repo` scope.
Claude Code uses your git credentials — if you can `git clone` the repo, Claude can install from it.

**Scopes for installation:**
- `--scope user` — available in all your projects (default)
- `--scope project` — available only in the current project directory

```bash
# Install globally for all your projects
claude plugins install my-plugin@my-repo --scope user

# Install only for the current project
claude plugins install my-plugin@my-repo --scope project
```

Project-scoped installs are tracked in `.claude/settings.json` (committed to git),
so your whole team gets the same plugin when they open the project.

---

## How the layered model works

There is no explicit "link" between layers. Claude Code simply loads components from all
active sources and merges them. The loading order determines priority.

### Loading order (highest priority last = wins)

```
Plugin agents/skills/commands
  └─► ~/.claude/agents/          (user-level overrides)
        └─► .claude/agents/      (project-level overrides)
```

**Same name = project wins.** If your plugin defines `code-reviewer.md` and your project
also has `.claude/agents/code-reviewer.md` — Claude uses the project version.

### How to extend, not override

The recommended pattern is **different names** at each layer:

```
Plugin:                          Project:
agents/code-reviewer.md          agents/dotnet-reviewer.md
agents/planner.md                agents/backend-dev.md
skills/api-design/               rules/backend.md  ← context, not agent
skills/tdd-workflow/             rules/frontend.md
```

The project-specific agents (like `backend-dev`) gain context from:
1. Their own system prompt (role + project-specific rules)
2. `.claude/rules/` files that load automatically when touching matching paths
3. `CLAUDE.md` which is always in context

### Practical example

```
Plugin provides:              Project adds:
─────────────────────         ──────────────────────────────
code-reviewer (generic)       rules/backend.md — .NET gotchas
planner (generic)             rules/frontend.md — Vue patterns
verifier (generic)            agents/backend-dev.md — .NET specialist
tdd-workflow skill            agents/frontend-dev.md — Vue specialist
pipeline command              pipeline-hotfix command — project-specific
```

When `backend-dev` agent runs, it automatically picks up `rules/backend.md` because
Claude loads path-scoped rules whenever it reads files matching those paths.
The agent system prompt stays lean; the rules carry the stack-specific context.

### Updating the plugin

```bash
# Pull latest version of your plugin
claude plugins update my-plugin@my-repo

# Or pin to a specific version in settings.json
"enabledPlugins": { "my-plugin@my-repo": "1.2.0" }
```

Project `.claude/` files are never touched by plugin updates — your customizations are safe.

---

## Minimal viable plugin checklist

To ship a working plugin from scratch:

- [ ] `agents/` — at least one agent with correct frontmatter
- [ ] `.claude-plugin/plugin.json` — name, version, description
- [ ] `.claude-plugin/marketplace.json` — registration metadata
- [ ] `package.json` — name, version, files list
- [ ] `README.md` — how to install and what's included
- [ ] Pushed to GitHub
- [ ] Installable via `claude plugins install github:user/repo`
