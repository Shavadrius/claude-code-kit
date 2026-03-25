# Multiple Pipelines — Different Workflows for Different Tasks

## The pattern

A pipeline is just a command file in `.claude/commands/`. You can have as many as you want.
Each one orchestrates agents differently depending on the task type.

```
.claude/commands/
├── pipeline.md              /pipeline       — full feature implementation
├── pipeline-bugfix.md       /pipeline-bugfix — fast bug fix loop
├── pipeline-review.md       /pipeline-review — review-only, no implementation
└── pipeline-refactor.md     /pipeline-refactor — refactor existing code
```

---

## When to create a separate pipeline

Create a new pipeline when:
- The **agent set** is different (e.g. bugfix doesn't need planner, goes straight to implementation)
- The **step order** is different (e.g. review-first for risky changes)
- The **verification scope** is different (e.g. quick smoke test vs full test suite)
- A specific workflow is used daily and deserves its own shortcut

---

## Example: full feature pipeline vs bugfix pipeline

### `/pipeline` — full feature
```
Plan → Implement (parallel if needed) → Review → Full verify
```
Use for: new features, non-trivial changes, anything touching multiple files

### `/pipeline-bugfix` — fast bug fix
```
Diagnose (read-only agent) → Fix (targeted implementation) → Verify (tests only)
```
Skips: planner (too slow for a 2-line fix), full review (code is small)
Use for: known bugs, test failures, small isolated fixes

### `/pipeline-review` — review only
```
Review → Report (no implementation)
```
Use for: reviewing a PR or a colleague's code, audit of existing functionality
Agents: code-reviewer only, nothing with write access

### `/pipeline-refactor` — refactor existing code
```
Analyze (read-only) → Plan → Refactor → Review → Verify
```
Key difference from feature pipeline: analysis phase reads the existing code first
and produces a "what exists and why" report before planning starts
Use for: large refactors, removing dead code, migrating patterns

---

## Pipeline command template

Each pipeline is a markdown file. Structure:

```markdown
# Pipeline: [Name]

## When to use this pipeline
[One sentence: what kind of task this is designed for]

## Step 1 — [Step name]
[Instructions for this step]
[Which agent(s) to spawn]
[What output to expect before proceeding]

## Step 2 — [Step name]
...

## Final output
[What to report when done]
```

---

## Choosing agents per pipeline

Not every pipeline needs every agent. Match agents to the task:

| Pipeline | Planner | Impl agents | Reviewer | Verifier |
|----------|---------|-------------|----------|----------|
| Feature | ✓ | ✓ (parallel) | ✓ | full suite |
| Bugfix | — | ✓ (single) | optional | tests only |
| Review-only | — | — | ✓ | — |
| Refactor | ✓ | ✓ | ✓ | full suite |
| Hotfix (prod) | — | ✓ | ✓ (mandatory) | smoke test |

---

## Naming conventions

Use a consistent prefix so CLI autocomplete groups them:

```
/pipeline           — default, most common
/pipeline-bugfix
/pipeline-review
/pipeline-refactor
/pipeline-hotfix
```

Or use a verb if that's more natural for your team:

```
/implement          — new feature
/fix                — bug
/review             — code review
/refactor           — refactor
```

---

## Reusing agent logic across pipelines

If multiple pipelines share the same review or verify step, don't copy-paste.
Reference the step by description: agents carry their own instructions via their definition in `.claude/agents/`.

The pipeline command only needs to say:
```markdown
## Step 3 — Review
Use the code-reviewer agent on all changed files.
```

The agent definition in `.claude/agents/code-reviewer.md` handles the rest.
This means updating an agent's behavior automatically updates all pipelines that use it.
