Review the current changes or the files listed in $ARGUMENTS.

## Step 1 — Code review

Spawn `code-reviewer` with the specified files (or all changed files if none specified).

## Step 2 — Security check (conditional)

If the changes touch authentication, authorization, input handling, or sensitive data:
- Spawn `security-reviewer` IN PARALLEL with `code-reviewer` (single message, two Agent tool calls)

## Output

Combined report from both reviewers with:
- Overall verdict
- All findings by severity
- Files reviewed
