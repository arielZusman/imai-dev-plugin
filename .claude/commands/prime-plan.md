---
description: Prime a session with the current plan and task context
---

# INPUTS
Optional `$ARGUMENTS`: task id (e.g., `T3`).

# STEPS
1. Show repo context:
   - `!git branch --show-current`
   - `!git status --short`
   - `!git log --oneline -5`
2. Read `docs/agent-state/plan-summary.md`.
3. If a task id is provided, read `docs/agent-state/task-<id>.md`.
   Otherwise, read the "Current Task" section from `plan-summary.md`.
4. Output a compact briefing with:
   - Current task goal
   - Required files
   - Verification command
   - Next action

# OUTPUT FORMAT
```
Session Context
- Branch: <name>
- Status: <clean|dirty>
- Recent commits: <list>

Task Brief
- ID: <id>
- Goal: <one sentence>
- Required files: <list>
- Verification: <command>
- Next action: <one step>
```
