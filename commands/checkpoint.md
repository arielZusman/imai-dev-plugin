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

**IMPORTANT:** Multi-file and single-file plans use DIFFERENT checkpoint locations:
- Multi-file: `docs/plans/YYYY-MM-DD-feature/checkpoint.md` (inside plan folder)
- Single-file: `docs/plans/.state/<plan-slug>.checkpoint.md` (in .state/ folder)

When this command is invoked:

1. **Parse the plan path** to derive checkpoint location:

   **If path contains `/intro.md` (multi-file subdirectory format):**
   - Plan folder: `docs/plans/2026-01-08-feature-name/`
   - Intro: `docs/plans/2026-01-08-feature-name/intro.md`
   - Checkpoint: `docs/plans/2026-01-08-feature-name/checkpoint.md` (in same folder)

   **If path ends with `.md` (single-file format):**
   - Plan: `docs/plans/2026-01-08-feature-name.md`
   - Slug: `2026-01-08-feature-name` (remove `.md`)
   - Checkpoint: `docs/plans/.state/2026-01-08-feature-name.checkpoint.md`

   **Key difference:** Multi-file stores checkpoint inside plan folder. Single-file uses `.state/` folder.

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
   - **Capture git state for conditional startup checks:**
     ```bash
     git rev-parse HEAD | cut -c1-8  # last_commit
     git status --porcelain | wc -l   # 0 = clean
     ```
   - **Generate continuation prompt** (see below)
   - **Enforce session stop** (one task per session policy - see Step 4)

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

5. **Write updated checkpoint** to derived path:
   - Multi-file: `docs/plans/<plan-folder>/checkpoint.md`
   - Single-file: `docs/plans/.state/<plan-slug>.checkpoint.md`

   **Multi-file format note:** The checkpoint file is the primary state store.
   - Task files (`task-N.md`) remain immutable after creation
   - The intro file's Execution Log is the secondary state for human readability
   - When updating status, also update the intro file's Execution Log table if multi-file format

6. **Verify checkpoint was persisted:**
   ```bash
   # Verify file exists and is readable
   cat <checkpoint-path> | head -20
   ```

   **If verification fails:**
   - Report: "ERROR: Checkpoint file not persisted correctly"
   - Retry write once
   - If still fails: STOP and report to user

7. **Output summary:**
   ```
   ✓ Checkpoint persisted: <checkpoint-path>
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

### Git State Section

Include git state for conditional startup checks in `/execute-plan`:

```yaml
git:
  last_commit: "a1b2c3d4"  # First 8 chars of HEAD
  working_tree_clean: true  # Based on git status --porcelain
  timestamp: "2026-01-15T10:30:00Z"
```

When `/execute-plan` resumes:
- If `working_tree_clean: true` AND `last_commit` matches current HEAD → skip `git status`
- Otherwise → run `git status` to detect uncommitted changes

### Continuation Prompt Section

Generate an exact prompt for the next session. Include in checkpoint:

```yaml
continuation:
  prompt: |
    Continue executing docs/plans/2026-01-14-feature.md

    Current position: Task 5 of 8
    Last completed: Task 4 - "Add validation service"

    Next task: Task 5 - "Write unit tests for validation"
    First step: Read src/services/validation.service.ts

    Handoff notes from Task 4:
    - ValidationService exports validateEmail and validatePassword
    - Uses RFC 5322 regex for email validation
    - Password requires 8+ chars, 1 number, 1 special char
  next_task:
    number: 5
    title: "Write unit tests for validation"
    first_step: "Read src/services/validation.service.ts"
```

**Benefits:**
- User can copy exact prompt to start fresh session
- Context is pre-loaded without re-reading plan
- Handoff notes carry forward essential information

## Example Workflow

**Single-file format:**
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
Creates checkpoint at: `docs/plans/.state/2026-01-08-auth-feature.checkpoint.md`

**Multi-file format (subdirectory):**
```
# Starting a task (use intro path)
/checkpoint docs/plans/2026-01-08-auth-feature/intro.md 3 started

# After completing the task
/checkpoint docs/plans/2026-01-08-auth-feature/intro.md 3 completed
```
Creates checkpoint at: `docs/plans/2026-01-08-auth-feature/checkpoint.md` (in same folder as intro)

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
