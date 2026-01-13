---
description: Save execution checkpoint for plan resumption across sessions
argument-hint: <plan-path> <task-number> <status>
arguments:
  plan_path:
    description: Path to the enriched plan file
    required: true
  task_number:
    description: Current task number
    required: true
  status:
    description: Task status (started, completed, error, blocked)
    required: true
---

# Checkpoint Command

Save or update a checkpoint file for the current plan execution.

## Usage

```
/checkpoint <plan_path> <task_number> <status>
```

**Status values:**
- `started` - Beginning work on task
- `completed` - Task finished successfully
- `error` - Error encountered (will prompt for notes)
- `blocked` - Task cannot proceed

## Instructions

<investigate_before_answering>
Read the existing checkpoint file before updating. Do not assume current state - verify actual task progress and session counts from the file.
</investigate_before_answering>

When this command is invoked:

1. **Parse the plan path** to derive checkpoint location:
   - Plan: `docs/plans/2026-01-08-feature-name.md`
   - Checkpoint: `docs/plans/.state/2026-01-08-feature-name.checkpoint.md`

2. **Read existing checkpoint** if it exists, otherwise create new one.

3. **Update based on status:**

   **If `started`:**
   - Set `execution.current_task` to task_number
   - Set `execution.current_phase` to `implement`
   - Add session_log entry: `task_started`

   **If `completed`:**
   - Run `npm run build` and `npm run test` to get actual results - do not ask user to describe status
   - Prompt for handoff notes (what the next task needs to know)
   - Add task to Completed Tasks section
   - Increment `tasks_completed_this_session`
   - Add session_log entry: `task_completed`
   - Check rotation heuristic

   **If `error`:**
   - Prompt for error description
   - Add session_log entry: `error`
   - Do NOT increment task count
   - Output: "Error logged. Fix the issue, then run `/checkpoint $ARGUMENTS.plan_path $ARGUMENTS.task_number completed`"

   **If `blocked`:**
   - Prompt for blocker description
   - Set `execution.status` to `blocked`
   - Add session_log entry: `blocked`
   - Output: "Task blocked. Resolve blocker before proceeding."

4. **Enforce session stop after task completion:**

   When status is `completed`:
   - Session MUST stop after this task
   - Output the pause message (see step 7)
   - Do NOT continue to next task in same session

   This is mandatory. One task per session ensures:
   - Fresh context for each task
   - Code review cannot be skipped
   - Checkpoint state is always current

5. **Write updated checkpoint** to `docs/plans/.state/<plan-slug>.checkpoint.md`

6. **Verify checkpoint was persisted:**
   ```bash
   # Verify file exists and is readable
   cat docs/plans/.state/<plan-slug>.checkpoint.md | head -20
   ```

   **If verification fails:**
   - Report: "ERROR: Checkpoint file not persisted correctly"
   - Retry write once
   - If still fails: STOP and report to user

7. **Output summary:**
   ```
   ✓ Checkpoint persisted: docs/plans/.state/<plan-slug>.checkpoint.md
   - Task: <task_number>
   - Status: <status>
   - Tasks this session: <count>
   ```

   **If status is `completed`:**
   ```
   ✓ Task [N] complete. Session will pause for context refresh.

   To continue: /execute-plan <plan-name>
   Next task: [N+1] - [title]
   ```

## Checkpoint File Format

See `docs/plans/.state/CHECKPOINT-FORMAT.md` for full specification.

## Example Workflow

```
# Starting a task
/checkpoint docs/plans/2026-01-08-auth-feature.md 3 started

# After completing the task
/checkpoint docs/plans/2026-01-08-auth-feature.md 3 completed

# If an error occurs
/checkpoint docs/plans/2026-01-08-auth-feature.md 3 error

# If blocked
/checkpoint docs/plans/2026-01-08-auth-feature.md 3 blocked
```

## Integration with Execution Workflow

Add checkpoint calls to the per-task workflow in your plan:

```markdown
**Checklist:**
- [ ] `/checkpoint <plan> <task> started`
- [ ] Implementation complete
- [ ] Verification passed
- [ ] Review passed
- [ ] Committed
- [ ] `/checkpoint <plan> <task> completed`
```
