---
description: Resume execution of an enriched plan with checkpoint support
arguments:
  - name: plan
    description: Plan filename (e.g., "influencer-tier-filter" or full path)
    required: true
  - name: task_id
    description: Optional task number to jump directly to (surgical resumption)
    required: false
---

Resume execution of an enriched implementation plan with checkpoint-aware context loading.

## Step 1: Locate Plan and Checkpoint

**Find the plan file:**
- If argument is a filename: search `docs/plans/*$ARGUMENTS.plan*.md`
- If argument is a path: use directly

**Check for checkpoint:**
- Derive checkpoint path: `docs/plans/.state/<plan-slug>.checkpoint.md`
- If checkpoint exists: use it for state and context
- If no checkpoint: fall back to scanning Execution Log in plan

## Step 2: Load State

**If checkpoint exists:**

1. Read checkpoint file
2. Extract current position:
   - `execution.current_task` - task number
   - `execution.current_phase` - where in task cycle
   - `execution.status` - in_progress/completed/blocked
3. Show recent session log (last 5 entries):
   ```
   Recent activity:
   - [timestamp] task_completed: Task 8 - Tests written
   - [timestamp] task_started: Task 9
   - [timestamp] error: Build failed - missing import
   ```
4. If `$ARGUMENTS.task_id` provided: override current_task with specified value

**If no checkpoint:**

1. Scan the Execution Log table in the plan
2. Find first task with status ⏳ Pending or 🔄 In Progress
3. If all tasks ✅ Done, check Final Review status

## Step 3: Load Task-Specific Context

**From checkpoint (preferred):**

Read the `Context Requirements` section for current task:
```yaml
context_requirements:
  task_9:
    required:
      - path: src/services/auth.service.ts
        sections: [validateToken, refreshToken]
    reference:
      - path: docs/auth-flow.md
```

**Re-read only required files**, not the entire codebase.

**From plan (fallback):**

Read the task's "Context Requirements" field and re-read listed files.

## Step 4: Verify Pre-Conditions

Before starting implementation:

1. **Check build status:**
   ```bash
   npm run build
   ```
   If fails: stop and report. Don't proceed with broken build.

2. **Check test status:**
   ```bash
   npm run test
   ```
   Compare against checkpoint's `last_verification.tests`.
   - If more failures than expected: investigate before proceeding
   - If matches expected: continue

3. **Check for uncommitted changes:**
   ```bash
   git status
   ```
   If dirty working tree: warn user, ask whether to continue or stash

## Step 5: Output Context Summary

Display to user:

```markdown
## Resuming: [Plan Name]

**Position:** Task [N] - [Title]
**Phase:** [implement/verify/review]
**Tasks completed this plan:** [X]/[Y]

### Recent Activity
[Last 5 session_log entries]

### Context Loaded
**Required files re-read:**
- src/services/auth.service.ts (validateToken, refreshToken)
- src/types/auth.types.ts

**Pre-conditions:**
- Build: ✅ passing
- Tests: 45/48 (3 expected failures)
- Working tree: clean

### Handoff from Previous Task
[handoff_notes from checkpoint for previous task]

### Ready to Execute
Task [N]: [Title]
First step: [first step from task]
```

## Step 6: Execute

Follow the **Execution Workflow** section in the plan. For each task:

1. `/checkpoint <plan> <task> started` (updates checkpoint)
2. Re-read context requirements
3. Implement the task steps
4. Run verification
5. Run code review per Review Protocol
6. Commit after review passes
7. `/checkpoint <plan> <task> completed` (updates checkpoint with handoff notes)

## Rotation Awareness

After loading checkpoint, check rotation heuristic:
- If `tasks_completed_this_session >= 4`: warn user
- If session has been running > 30 min: warn user

Output:
```
⚠️ Rotation recommended: [reason]
Consider clearing session after completing current task.
```

## Error Handling

**If checkpoint is corrupted/invalid:**
- Fall back to Execution Log in plan
- Warn: "Checkpoint invalid, falling back to plan state"

**If plan file not found:**
- List available plans in `docs/plans/`
- Ask user to specify correct path

**If task_id out of range:**
- Show available tasks
- Ask user to specify valid task number

## Examples

**Basic resume:**
```
/resume-plan hashtag-budget-strategy
```
Loads checkpoint if exists, resumes from current position.

**Jump to specific task:**
```
/resume-plan hashtag-budget-strategy 9
```
Loads checkpoint but overrides to start at Task 9.

**Resume with full path:**
```
/resume-plan docs/plans/2026-01-08-hashtag-budget-strategy.md
```
Uses exact path provided.
