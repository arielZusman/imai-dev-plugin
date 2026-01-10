---
description: Orchestrated plan execution with selective sub-agent dispatch
argument-hint: <plan-name> [tasks]
arguments:
  - name: plan
    description: Plan filename or path (e.g., "hashtag-budget-strategy" or full path)
    required: true
  - name: tasks
    description: "Optional: specific tasks to execute (e.g., '3' or '3-5' or '3,5,7')"
    required: false
---

# Execute Plan Command

Orchestrated execution of an enriched implementation plan. Runs in main session, selectively dispatches complex tasks to sub-agents.

## Overview

This command implements **lean orchestration**:
- Simple tasks execute directly in main session (low overhead)
- Complex tasks dispatch to specialist sub-agents (context isolation)
- Parallel-eligible tasks can run concurrently via multiple sub-agents
- Checkpoint updates after each task
- Rotation recommendations based on task count/time

## Step 1: Load Plan and State

**Locate files:**
```
Plan: docs/plans/*$ARGUMENTS.plan*.md
Checkpoint: docs/plans/.state/<plan-slug>.checkpoint.md
```

**Parse plan for:**
- Task list with metadata (complexity, dispatch hint, dependencies)
- Orchestration Hints section (parallel groups, dispatch decisions)
- Current execution state from checkpoint (or Execution Log fallback)

**Determine task scope:**
- If `$ARGUMENTS.tasks` provided: execute only specified tasks
- Otherwise: execute from current position to end (respecting dependencies)

## Step 2: Pre-Flight Verification

Before starting any task:

```bash
# Verify build passes
npm run build

# Verify test baseline
npm run test

# Check working tree
git status
```

**If pre-flight fails:**
- Report failure with details
- Do NOT proceed
- Suggest: "Fix issues and re-run `/execute-plan`"

## Step 3: Task Execution Loop

For each task in scope (respecting dependency order):

### 3.1 Evaluate Dispatch Decision

Check task's `Dispatch` field or use default rules:

| Condition | Decision | Rationale |
|-----------|----------|-----------|
| `Dispatch: direct` in task | Execute directly | Explicit instruction |
| `Dispatch: sub-agent (X)` in task | Dispatch to agent X | Explicit instruction |
| Complexity 🟢 + < 50 lines | Execute directly | Low overhead wins |
| Complexity 🟡/🔴 | Dispatch to sub-agent | Context isolation benefit |
| TDD test task | Dispatch to test-writer | Fresh context for test design |
| Implementation after tests | Dispatch to implementer | Isolation from test context |

### 3.2 Execute Task

**If executing directly:**

1. Run `/checkpoint <plan> <task> started`
2. Read task's Context Requirements (required files)
3. Execute task steps as written in plan
4. Run task's Verify step
5. Run code review: `/pr-review-toolkit:review-pr <scope>`
6. Fix critical issues (max 2 iterations)
7. Commit with descriptive message
8. Run `/checkpoint <plan> <task> completed`
9. Provide handoff notes when prompted

**If dispatching to sub-agent:**

1. Run `/checkpoint <plan> <task> started`
2. Build task prompt (see "Sub-Agent Prompt Template" below)
3. Dispatch via Task tool:
   ```
   Task tool:
   - subagent_type: general-purpose (or specific agent if named)
   - prompt: [built prompt with task details]
   - description: "Execute Task N: [title]"
   ```
4. Wait for sub-agent completion
5. Verify sub-agent results:
   - Check files were modified as expected
   - Run verification step
   - Run code review
6. If verification fails: retry once with feedback, then mark blocked
7. Commit changes
8. Run `/checkpoint <plan> <task> completed`

### 3.3 Handle Parallel Tasks

If Orchestration Hints indicates parallel group:

```yaml
parallel_groups:
  - [task_2, task_4]  # Can run simultaneously
```

**For parallel execution:**
1. Verify all dependencies complete
2. Build prompts for each task in group
3. Dispatch ALL tasks in single message (parallel Task tool calls):
   ```
   <Task tool call 1: Task 2>
   <Task tool call 2: Task 4>
   ```
4. Wait for all to complete
5. Verify each task's results
6. Checkpoint each task

**Note:** True parallelism requires multiple Task tool calls in same message.

## Step 4: Rotation Management

After each task completion, check rotation heuristic:

**Triggers:**
- `tasks_completed_this_session >= 4`
- Elapsed time > 30 minutes since session start

**If triggered:**
```
⚠️ Rotation recommended after [N] tasks.

Options:
1. Continue with current task (not recommended)
2. Pause execution, save checkpoint, rotate session

To resume after rotation:
/resume-plan <plan-path>
```

**If user continues:** proceed but warn again after next task.

## Step 5: Completion

When all tasks in scope complete:

1. Run final verification:
   ```bash
   npm run test
   npm run build
   ```

2. If Final Review not done:
   - Run `/pr-review-toolkit:review-pr all`
   - Update Final Review status in plan

3. Output summary:
   ```
   ## Execution Complete

   **Plan:** [name]
   **Tasks completed:** [list]
   **Final verification:** build ✅, tests ✅

   Next steps:
   - [ ] Review changes: `git diff main`
   - [ ] Create PR or merge
   ```

---

## Sub-Agent Prompt Template

When dispatching to sub-agent, build this prompt:

```markdown
# Task [N]: [Title]

You are executing a task from an implementation plan. Complete ONLY this task.

## Context

**Plan:** [plan name]
**Your task:** [N] of [total]
**Complexity:** [🟢/🟡/🔴]

## Files to Read First

These files contain context you need:
- `[path]` - [what to look for]
- `[path]` - [what to look for]

## Task Details

[Copy task's Steps section from plan]

## Verification

After implementation, verify:
[Copy task's Verify section]

## Constraints

- Modify ONLY files listed in this task
- Do NOT proceed to other tasks
- If blocked, report what's blocking and stop

## Expected Output

When complete, report:
1. Files modified: [list]
2. Verification result: [pass/fail]
3. Any issues encountered
4. Handoff notes for next task
```

---

## Dispatch Decision Examples

**Task: Add interface definitions (🟢 Simple)**
```
Dispatch: direct
Rationale: < 20 lines, no complex logic
```

**Task: Write unit tests for auth service (🟡 Moderate)**
```
Dispatch: sub-agent (general-purpose)
Rationale: TDD test writing benefits from fresh context
```

**Task: Implement two-phase allocation algorithm (🔴 Complex)**
```
Dispatch: sub-agent (general-purpose)
Rationale: Complex implementation, isolate from planning context
```

**Task: Update environment variables (🟢 Simple)**
```
Dispatch: direct
Rationale: Config change, trivial
```

---

## Error Handling

**Sub-agent returns error:**
1. Log error to checkpoint session_log
2. Attempt retry with error context (once)
3. If still fails: mark task blocked, stop execution
4. Report: "Task [N] blocked. Manual intervention required."

**Verification fails after implementation:**
1. If direct execution: fix in place (max 2 cycles)
2. If sub-agent: dispatch again with failure context
3. After 2 failures: mark blocked

**Dependency not met:**
- Skip task, continue with others if possible
- Report: "Task [N] skipped - dependency Task [M] not complete"

**Build/test regression:**
- Stop execution immediately
- Report which task caused regression
- Do NOT proceed

---

## Examples

**Execute entire plan from current position:**
```
/execute-plan hashtag-budget-strategy
```

**Execute specific task:**
```
/execute-plan hashtag-budget-strategy 5
```

**Execute task range:**
```
/execute-plan hashtag-budget-strategy 3-7
```

**Execute specific tasks (non-contiguous):**
```
/execute-plan hashtag-budget-strategy 2,5,8
```

---

## Integration with Other Commands

- **Before execution:** `/resume-plan` to load context and verify state
- **During execution:** `/checkpoint` called automatically
- **After tasks:** Rotation check, may recommend session clear
- **After completion:** Suggest `/pr-review-toolkit:review-pr` for final review
