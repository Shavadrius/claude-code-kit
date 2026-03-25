Run the full development pipeline for the task described in $ARGUMENTS.

## Step 1 — Plan

Spawn the `planner` agent with the task description.

Do NOT proceed to Step 2 until the planner has produced a written plan with affected files and change order.

## Step 2 — Implement

Based on the plan:
- Single domain → spawn one `implementer` agent
- Multiple independent domains → spawn implementer agents IN PARALLEL (single message, multiple Agent tool calls)

Wait for all implementers to complete before proceeding.

## Step 3 — Review

Spawn `code-reviewer` with the list of changed files.

If verdict is REQUEST_CHANGES or BLOCKER:
- Fix the issues with `implementer`
- Re-run `code-reviewer`
- Repeat until verdict is APPROVE

## Step 4 — Verify

Spawn `verifier`.

If status is FAIL:
- Spawn `debugger` with the failure output
- Spawn `implementer` with the debugger's proposed fix
- Re-run `verifier`
- Repeat until status is PASS

## Step 5 — Final report

Output:
- Summary of changes made
- Reviewer verdict
- Verifier status
- List of modified files
