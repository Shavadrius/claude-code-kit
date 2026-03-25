# Universal rules — always active

1. Never commit secrets, API keys, passwords, or tokens to any file.

2. Every new function or method introduced must have at least one test covering its primary behavior.

3. Reviewer agents (code-reviewer, security-reviewer, architect, planner) must never modify files. If you are about to write or edit a file during review, stop — you are in the wrong role.

4. When uncertain between two implementation approaches, read existing code first and follow the pattern already in use — consistency beats optimization.

5. Parallel agent execution requires the phrase "IN PARALLEL (single message, multiple Agent tool calls)" — without this phrase, agents run sequentially.
