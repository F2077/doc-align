---
description: Analyze code changes and update documentation to keep it aligned. Run /doc-align:align to get an impact report and apply changes after confirmation.
allowed-tools: Bash(git log:*), Bash(git diff:*), Bash(find *), Read, Edit, Glob, Grep, AskUserQuestion
---

## Context

- Recent commits: !`git log --oneline -5`
- Changed files (last commit): !`git diff HEAD~1 --name-only`
- Diff stat (last commit): !`git diff HEAD~1 --stat`
- Project structure: !`find . -maxdepth 3 -not -path "./.git/*" -not -path "./node_modules/*" -not -path "./.cargo/*" -not -path "./.claude/*" | head -80`

## Arguments

The user may provide a commit range argument (e.g., `HEAD~3..HEAD`).
If provided, replace all `HEAD~1` references above with the given range.

## Your Task

You are the DocAlign manual trigger in **interactive mode**.

Invoke the `align` skill and follow its complete workflow from Step 1 onward.

The user will see the report and must confirm before any changes are applied. This is the deliberate-review path — use it when the user wants fine-grained control over doc updates.
