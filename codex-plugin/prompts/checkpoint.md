---
description: Save or update execution checkpoint for plan resumption
argument-hint: PLAN_PATH=<path> TASK_NUMBER=<n> STATUS=<started|completed|error|blocked>
---

Update the checkpoint file for the plan.

1) Derive checkpoint path:
   - Plan: docs/plans/2026-01-08-feature.md
   - Checkpoint: docs/plans/.state/2026-01-08-feature.checkpoint.md

2) Read existing checkpoint if present, otherwise create a new one using
   the format below.

3) Update based on STATUS:

- started:
  - execution.current_task = TASK_NUMBER
  - execution.current_phase = implement
  - add session_log entry: task_started

- completed:
  - run build/test if applicable (use plan verification commands if listed;
    otherwise run npm run build and npm run test when package.json exists)
  - prompt for handoff notes
  - add task to Completed Tasks
  - increment tasks_completed_this_session
  - add session_log entry: task_completed
  - output pause message: one task per session

- error:
  - prompt for error description
  - add session_log entry: error

- blocked:
  - prompt for blocker description
  - execution.status = blocked
  - add session_log entry: blocked

4) Write the checkpoint and verify it exists.

Checkpoint format (create if missing):

# Checkpoint: <plan title>

## Execution
- current_task: <n>
- current_phase: <implement|verify|review>
- status: <in_progress|blocked|completed>
- tasks_completed_this_session: <n>

## Completed Tasks
- Task <n>: <title>

## Handoff Notes
- Task <n>: <notes>

## Session Log
- timestamp: <ISO-8601>
  event: task_started|task_completed|error|blocked
  task: <n>
  notes: <optional>
