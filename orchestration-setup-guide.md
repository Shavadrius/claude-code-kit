# Setting Up Agent Orchestration with Claude Code

A step-by-step methodology guide. Technology-agnostic.
Based on official Claude Code docs + community best practices (shanraisshan/claude-code-best-practice, Boris Cherny, Thariq Shihipar).

---

## The mental model

Claude Code has five distinct primitives. Know the difference before building anything:

| Primitive | Location | What it is | Who triggers it |
|-----------|----------|------------|-----------------|
| **CLAUDE.md** | `./CLAUDE.md`, `~/.claude/CLAUDE.md` | Persistent instructions, always loaded | Auto (session start) |
| **Rules** | `.claude/rules/*.md` | Path-scoped context, loaded on demand | Auto (when Claude reads matching files) |
| **Commands** | `.claude/commands/<name>.md` | Prompt template injected into current context | Human (`/command-name`) |
| **Agents** | `.claude/agents/<name>.md` | Isolated subagent with own context, tools, model | Claude auto-delegates, or `@agent-name`, or CLI |
| **Skills** | `.claude/skills/<name>/SKILL.md` | Like commands but structured: can fork context, preload, inject shell output | Human or Claude (configurable) |

**Core distinction:**
- Commands and skills = **knowledge injected into existing context**
- Agents = **fresh isolated context**, the agent never sees your main conversation

---

## Step 1 — Design before you build

Answer these questions before creating any files:

**1. What are your domains?**
Group by independent areas of responsibility (e.g. backend, frontend, infrastructure, docs).
Each domain is a candidate for its own agent.

**2. What tasks repeat daily?**
If you do something more than once a day, it belongs in a command or skill.
If you do it once per feature, it belongs in a pipeline.

**3. What needs isolation?**
Any task where author bias is a problem (review, QA, security audit) must run in a separate context.
Give these agents `disallowedTools: Write, Edit` — they physically cannot modify files.

**4. What can run in parallel?**
Independent domains (backend + frontend) can run simultaneously.
Dependent steps (review after implementation) must be sequential.

**Rule of thumb:** 4–6 agents covers most projects. More than that and you're over-engineering.

---

## Step 2 — Write CLAUDE.md correctly

CLAUDE.md is loaded every session — treat its length like RAM.

**Hard limit: 200 lines.** Past that, Claude starts ignoring content.

**What belongs in CLAUDE.md:**
- Build, test, and run commands (so `run the tests` works on first try, no setup needed)
- High-level architecture in one paragraph
- Non-obvious project conventions
- Links to external context with `@path/to/file` imports

**What does NOT belong in CLAUDE.md:**
- Domain-specific rules (move to `.claude/rules/`)
- Step-by-step workflows (move to commands or skills)
- Everything you'd find in a README

**Trick:** wrap critical rules in `<important if="...">` tags — Claude is more likely to follow tagged rules as files grow longer.

**Test:** a new developer should be able to open the project, say "run the tests" to Claude, and it should work on the first try. If not, CLAUDE.md is missing something.

---

## Step 3 — Create path-scoped rules

Rules reduce agent system prompt size. Instead of repeating context in every agent, put it in rules — Claude loads them automatically when touching matching files.

```yaml
---
paths:
  - "src/api/**"
  - "tests/integration/**"
---

# Rules that apply when touching these paths
- Convention 1
- Convention 2
```

**What goes in rules vs agent system prompts:**

| Rules | Agent system prompt |
|-------|---------------------|
| Project conventions (naming, patterns) | Role and responsibility |
| Framework gotchas | Workflow steps (TDD, etc.) |
| Shared schema / API contracts | Tool and model selection |
| Things true regardless of who's working | Agent-specific checklists |

Rules without `paths:` load always — use them sparingly, they consume context on every session.

---

## Step 4 — Create agent definition files

Each agent is `.claude/agents/<name>.md` with YAML frontmatter + system prompt.

### Minimal template

```yaml
---
name: agent-name
description: Trigger condition — when should Claude spawn this agent? Write for the model, not for humans.
tools: Read, Write, Edit, Bash, Grep, Glob
model: sonnet
---

System prompt: role, context, rules, workflow.
```

### Key frontmatter fields

| Field | Notes |
|-------|-------|
| `name` | Slug, used for `@mention` and CLI |
| `description` | **Most critical field** — Claude reads this to decide auto-delegation |
| `tools` | Whitelist what the agent CAN use |
| `disallowedTools` | Explicit deny — always wins over `tools` |
| `model` | `sonnet` for implementation, `opus` for planning/review, `haiku` for cheap/fast tasks |
| `permissionMode` | `default`, `acceptEdits`, `dontAsk` |
| `maxTurns` | Prevent runaway agents on complex tasks |

### Tool allocation by agent role

```yaml
# Implementation agents — full access
tools: Read, Write, Edit, Bash, Grep, Glob

# Review / analysis — read only, enforces no author bias
tools: Read, Grep, Glob
disallowedTools: Write, Edit, Bash

# Verification — can run tests but not modify code
tools: Read, Bash, Grep, Glob
disallowedTools: Write, Edit
```

### Writing the description field

The description is a trigger condition, not a job title. Write it as "when should I use this":

```yaml
# Bad — Claude doesn't know when to use this
description: Backend developer

# Good — Claude knows exactly when to spawn this agent
description: Use when implementing API endpoints, database schema changes, or business logic. Writes tests before code.

# For review agents — add "proactively" to make Claude always run it
description: Use proactively after any code changes. Runs in a separate context with no author bias.
```

### System prompt principles

- **Give goals and constraints, not prescriptive steps** — don't railroad Claude
- **Focus on what pushes Claude out of its default behavior** — don't state the obvious
- **Add a Gotchas section** — highest-signal content, list failure modes you've discovered
- Keep it short: rules carry the domain context, the system prompt carries the role

---

## Step 5 — Create skills (advanced commands)

Skills are more powerful than commands. Use them when you need:
- Context isolation (`context: fork` — main context only sees the final result)
- Dynamic shell output injected at invocation time
- Reusable scripts or example files alongside the instructions
- Fine-grained tool restrictions

```
.claude/skills/my-skill/
├── SKILL.md          # Required: instructions + YAML frontmatter
├── template.md       # Optional: output template
├── examples/         # Optional: example outputs
└── scripts/          # Optional: scripts Claude can execute
```

```yaml
---
name: my-skill
description: When to use this skill — written as a trigger condition
context: fork        # Run in isolated subagent, main context sees only the result
allowed-tools: Read, Grep
---

Instructions...

# Gotchas
- Known failure mode 1
- Known failure mode 2
```

**Inject live shell output into the prompt:**
```markdown
Current git status: !`git status --short`
Current branch: !`git branch --show-current`
```

Claude sees the command output at invocation time — not stale cached data.

**When to use a skill vs command:**

| Use a command | Use a skill |
|---------------|-------------|
| Simple workflow, 1 context | Needs context isolation |
| No dynamic data needed | Needs live shell output |
| Human-triggered only | Can be auto-triggered by Claude |
| No supporting files | Has templates, scripts, examples |

---

## Step 6 — Build the pipeline command

The pipeline is a human-invoked command (`/pipeline`) that orchestrates agents in the right order.

### Pipeline template

```markdown
Run the full development pipeline for the task in the argument.

## Step 1 — Plan
Before writing any code:
- Identify which domains are affected
- Map file change dependencies
- Flag project-specific risks

Output a plan. Do NOT proceed without one.

## Step 2 — Implement
Based on the plan:
- Single domain → one agent
- Multiple independent domains → spawn IN PARALLEL (single message, multiple Agent tool calls)

Wait for all agents to complete.

## Step 3 — Review
Spawn the code-reviewer agent on all changed files.
Fresh context — it has not seen the implementation.

- "Request changes" → fix, re-run reviewer
- "Approve" → proceed

## Step 4 — Verify
Spawn the verifier agent.
All checks must pass before done.

## Final report
- What was implemented
- Review verdict
- Verification status
- Files changed
```

### The parallel execution pattern

When the pipeline needs two agents simultaneously, the instruction must be explicit:

```markdown
spawn backend-dev and frontend-dev IN PARALLEL (single message, two Agent tool calls)
```

Without this phrasing, the orchestrator will run them sequentially. The phrase "single message, two Agent tool calls" is the signal for the orchestrator model to emit both tool calls in one response.

---

## Step 7 — Configure permissions correctly

Permissions have a clear hierarchy. More specific always wins:

```
Managed (system admin) → User (~/.claude/settings.json) → Project (.claude/settings.json) → Local (.claude/settings.local.json)
```

**Use wildcard syntax, not broad grants:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Edit(src/**)",
      "Bash(git commit*)",
      "Bash(docker compose*)"
    ],
    "defaultMode": "default"
  }
}
```

Avoid `bypassPermissions` or `Bash(*)` in shared settings — it defeats the safety model.
Use `settings.local.json` (gitignored) for personal broad permissions on your own machine.

**Per-agent permissions:**
`permissionMode` in agent frontmatter overrides project settings for that agent only.
Reviewer agents should never need `Write` or `Edit` permissions — enforce via `disallowedTools`.

---

## Step 8 — Add hooks for automation

Hooks run outside the agentic loop on specific events. Use them for automation that should always happen regardless of what Claude does.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [{"type": "command", "command": "npm run format --silent"}]
      }
    ],
    "Stop": [
      {
        "hooks": [{"type": "command", "command": "./scripts/verify-complete.sh"}]
      }
    ]
  }
}
```

**Most useful hooks:**

| Hook | Use case |
|------|----------|
| `PostToolUse(Edit)` | Auto-format after every file edit — fixes the last 10% Claude gets wrong |
| `PreToolUse(Bash)` | Block destructive commands, validate before running |
| `Stop` | Nudge Claude to verify its work, run final checks |
| `SubagentStop` | Collect results from subagents into a shared log |

---

## Step 9 — Test and iterate

### Smoke test sequence

1. **Single agent** — invoke directly: `@backend-dev add a health endpoint`
   Check: does it follow project conventions? Does it write tests first?

2. **Reviewer** — invoke on existing code: `@code-reviewer review src/auth/`
   Check: can it NOT edit files? Does it output structured severity levels?

3. **Full pipeline** — `\pipeline add a simple health check endpoint`
   Check: does it plan before implementing? Does parallel invocation actually happen?

### Warning signs

| Symptom | Fix |
|---------|-----|
| Agent doesn't auto-trigger | Rewrite `description` — it's the primary signal |
| Agent edits files it shouldn't | Add `disallowedTools: Write, Edit` |
| Pipeline skips steps | Add "Do NOT proceed until X is complete" |
| Context overflow on large tasks | Use `context: fork` in skills, break tasks smaller |
| Parallel agents conflict | Their domains are not truly independent — run sequentially |
| Claude ignores CLAUDE.md rules | Add `<important if="...">` tags, or move to rules with path scoping |

### The "dumb zone"

When context fills past ~50%, quality degrades. Run `/compact` manually at that point.
Use `/clear` to reset context when switching to a completely different task.
Commit often — at least once per task completion, ideally every hour.

---

## Step 10 — What to commit vs gitignore

```gitignore
# Personal only — gitignore
.claude/settings.local.json
.claude/worktrees/

# Team shared — commit everything else
# .claude/agents/     ← agent definitions
# .claude/rules/      ← path-scoped rules
# .claude/commands/   ← pipeline and daily commands
# .claude/skills/     ← reusable skill workflows
# .claude/settings.json ← shared permissions
# CLAUDE.md
```

---

## The universal development workflow

All major community frameworks converge on the same pattern:

```
Research → Plan → Execute → Review → Ship
```

In Claude Code terms:
```
Explore agent (read-only)
  └─► Planner agent (read-only, outputs structured plan)
        └─► Implementation agents (parallel if independent)
              └─► Reviewer agent (fresh context, read-only)
                    └─► Verifier agent (runs tests, no edits)
```

The key insight: **separate context windows make results better.**
One agent causes bugs. A different agent (same model, fresh context) finds them.
This is test-time compute — you're spending more tokens to get a better outcome.

---

## Open questions (no consensus yet)

These are genuinely unsolved — the community is still figuring them out:

- When should you use a command vs agent vs skill vs vanilla Claude?
- How often should you update agents/commands as models improve?
- Does giving a subagent a detailed persona actually improve output quality?
- Should you use built-in plan mode, or build your own planning workflow?
- Can you maintain a spec for every feature and regenerate code from specs?
