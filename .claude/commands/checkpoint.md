---
description: Save execution checkpoint for plan resumption across sessions
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
   - Prompt for verification results (build status, test status)
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

4. **Check rotation heuristic:**
   - If `tasks_completed_this_session >= 4`:
     Output warning: "Rotation recommended: 4+ tasks completed. Consider clearing session and running `/resume-plan $ARGUMENTS.plan_path`"
   - Calculate elapsed time since session_start
   - If elapsed > 30 minutes:
     Output warning: "Rotation recommended: Session > 30 min. Consider clearing session and running `/resume-plan $ARGUMENTS.plan_path`"

5. **Write updated checkpoint** to `docs/plans/.state/<plan-slug>.checkpoint.md`

6. **Output summary:**
   ```
   Checkpoint saved: docs/plans/.state/<plan-slug>.checkpoint.md
   - Task: <task_number>
   - Status: <status>
   - Tasks this session: <count>
   - Rotation: <recommended|not needed>
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
