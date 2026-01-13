---
description: Execute an enriched plan task-by-task with checkpoints
argument-hint: PLAN=<plan-name-or-path> [ TASKS=<n|n-m|n,m> ]
---

Execute tasks from an enriched plan in docs/plans/. This workflow is single-agent
only (Codex does not support sub-agents), so execute tasks directly and sequentially.

Plan lookup:
- If PLAN is a path, use it.
- Otherwise, resolve docs/plans/*<PLAN>*.md.

Checkpoint lookup:
- docs/plans/.state/<plan-slug>.checkpoint.md

Pre-flight checks (before any task):
1) pwd (confirm correct repo)
2) If package.json exists: npm run build
3) If package.json exists: npm run test
4) git status

If any pre-flight check fails, stop and report.

Task execution (one task per session):
- Load checkpoint state (if any) and determine current task.
- If TASKS is provided, execute only those tasks in order.
- For each task:
  1) /prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=started
  2) Re-read the task's Context Requirements
  3) Search-first: confirm the feature/fix is not already implemented
  4) Implement the task steps
  5) Run the task's Verify command
  6) Review changes with git diff --staged and git status
  7) Commit with a descriptive message
  8) /prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=completed
  9) STOP. Do not continue to the next task in the same session.

If verification fails, fix the issue (max 2 cycles), then proceed. If blocked,
run /prompts:checkpoint with STATUS=blocked and stop.
