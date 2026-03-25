Fix the bug described in $ARGUMENTS using a minimal, targeted approach.

## Step 1 — Diagnose

Spawn `debugger` with the bug description.

Do NOT proceed until root cause is identified with evidence.

## Step 2 — Fix

Spawn `implementer` with the debugger's proposed fix.

If implementer touches more than 5 files: STOP. Report that the fix scope is too large for pipeline-bugfix — use `/pipeline` instead.

## Step 3 — Verify

Spawn `verifier`.

If FAIL: repeat from Step 1 with the new failure output.

## Output

- Root cause identified
- Fix applied
- Verification status
