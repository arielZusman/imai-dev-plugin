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

## Directory Safety

Before running `npm`, `ng`, or package manager commands, verify the correct directory:
```bash
pwd
cd <plan-target-directory>  # if needed
```

Running commands in the wrong directory corrupts the wrong `package.json`.

---

## Overview

This command implements **lean orchestration**:
- Simple tasks execute directly in main session (low overhead)
- Complex tasks dispatch to specialist sub-agents (context isolation)
- Parallel-eligible tasks can run concurrently via multiple sub-agents
- Checkpoint updates after each task (MANDATORY - do not skip)
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

Before starting any task, run ALL of these checks:

```bash
# 1. Verify correct directory (CRITICAL)
pwd  # Must match plan's target directory
cd <plan-target-directory>  # If not already there

# 2. Verify build passes
npm run build

# 3. Verify test baseline (establishes what should pass)
npm run test

# 4. Check working tree is clean
git status
```

**Output a status summary:**
```
Pre-flight check:
- Directory: ✅ /path/to/target
- Build: ✅ passed
- Tests: ✅ X passed, Y skipped
- Git: ✅ clean working tree
```

**If ANY pre-flight step fails:**
- Report failure with details
- Do NOT proceed
- Suggest: "Fix issues and re-run `/execute-plan`"

## Step 3: Task Execution Loop

For each task in scope (respecting dependency order):

### 3.1 Evaluate Dispatch Decision

<investigate_before_answering>
Read the task's actual Dispatch field before deciding. Do not assume dispatch mode based on task title alone.
</investigate_before_answering>

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

Use this checklist for every task:

```
Task [N] Checklist:
□ CHECKPOINT: /checkpoint <plan> <task> started
□ DIRECTORY: pwd shows correct target directory
□ IMPLEMENT: Execute task steps
□ VERIFY: Run task's verification command
□ REVIEW: /pr-review-toolkit:review-pr staged
□ FIX: Address critical issues (max 2 iterations)
□ COMMIT: git commit with descriptive message
□ CHECKPOINT: /checkpoint <plan> <task> completed
```

<mandatory_code_review>
Code review after each task is not optional. Reviews catch issues before they compound across tasks.
Run `/pr-review-toolkit:review-pr staged` before committing.
</mandatory_code_review>

**If executing directly:**

1. Run `/checkpoint <plan> <task> started`
2. Verify directory with `pwd` - must be in plan's target directory
3. Read task's Context Requirements (required files)
4. Execute task steps as written in plan
5. Run task's Verify step
6. Run `/pr-review-toolkit:review-pr staged` (see mandatory_code_review above)
7. Fix critical issues (max 2 iterations)
8. Commit with descriptive message
9. Run `/checkpoint <plan> <task> completed`
10. Provide handoff notes when prompted

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
   - Run code review (see mandatory_code_review above)
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

Rotation prevents context degradation after sustained work. As context fills, attention to recent instructions decreases and error rates increase.

**Track progress in your todo list with task count:**
```
Example todo list format:
- [x] Task 1: Update dependencies (1/6)
- [x] Task 2: Fix imports (2/6)
- [x] Task 3: Update config (3/6)
- [ ] Task 4: Run migrations (4/6) ⚠️ ROTATION RECOMMENDED
- [ ] Task 5: Update tests (5/6)
- [ ] Task 6: Final verification (6/6)
```

**After EACH task completion, check rotation heuristic:**

**Triggers (check BOTH):**
- `tasks_completed_this_session >= 4` → Output rotation warning
- Elapsed time > 30 minutes since session start → Output rotation warning

**If triggered, you MUST output:**
```
⚠️ ROTATION RECOMMENDED

Tasks completed this session: [N]
Time elapsed: [X] minutes

Continuing without rotation may lead to:
- Context degradation
- Increased error rate
- Missed instructions

Options:
1. Continue (not recommended)
2. Pause and rotate: /checkpoint <plan> <task> completed, then start new session

To resume: /resume-plan <plan-path>
```

**If user continues:** proceed but warn again after EVERY subsequent task.

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

## Handling Pre-Commit Hooks

When committing, pre-commit hooks may fail. Follow these guidelines:

**When bypass is ACCEPTABLE (`--no-verify`):**
- Pre-existing lint errors not introduced by your changes
- Formatting issues from automated tools (prettier, eslint --fix)
- Hook runs checks unrelated to your changes (e.g., unmodified files)

**When bypass is NOT acceptable:**
- New lint errors in files you modified
- Type errors in your code
- Failing tests
- Security vulnerabilities flagged by hooks

**Decision flow:**
```
Hook failed → Check if error is in files YOU modified
  ├── Yes → Fix the issue, do NOT bypass
  └── No → Ask user: "Pre-commit hook failed on pre-existing issues. Bypass with --no-verify?"
```

**If user approves bypass:**
```bash
git commit --no-verify -m "your message"
```

**Document bypasses:** Add note to Execution Log: "Committed with --no-verify due to [reason]"

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
