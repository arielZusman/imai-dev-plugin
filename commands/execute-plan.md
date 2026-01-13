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

**If checkpoint exists (resuming):**

1. Extract current position:
   - `execution.current_task` - task number
   - `execution.current_phase` - where in task cycle
   - `execution.status` - in_progress/completed/blocked

2. Display recent activity (last 5 session_log entries):
   ```
   Recent activity:
   - [timestamp] task_completed: Task 8 - Tests written
   - [timestamp] task_started: Task 9
   - [timestamp] error: Build failed - missing import
   ```

3. Show handoff notes from previous task (if any)

4. Check rotation heuristics:
   - If `tasks_completed_this_session >= 4`: warn user
   - If session has been running > 30 min: warn user
   ```
   ⚠️ Rotation recommended: [reason]
   Consider clearing session after completing current task.
   ```

## Step 1.5: Load Task-Specific Context

<investigate_before_answering>
Read the actual Context Requirements from the checkpoint or plan. Do not guess file paths or assume what context is needed.
</investigate_before_answering>

**From checkpoint (preferred):**

Read the `context_requirements` section for current task:
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

**Output context summary:**
```markdown
## Resuming: [Plan Name]

**Position:** Task [N] - [Title]
**Phase:** [implement/verify/review]
**Tasks completed this plan:** [X]/[Y]

### Context Loaded
**Required files re-read:**
- src/services/auth.service.ts (validateToken, refreshToken)
- src/types/auth.types.ts

### Handoff from Previous Task
[handoff_notes from checkpoint for previous task]

### Ready to Execute
Task [N]: [Title]
First step: [first step from task]
```

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
□ SEARCH FIRST: Verify feature/fix doesn't already exist (see search-first below)
□ IMPLEMENT: Execute task steps
□ VERIFY: Run task's verification command
□ BUILD: npm run build (must pass)
□ REVIEW: /pr-review-toolkit:review-pr staged (MANDATORY - do not skip)
□ FIX: Address critical issues (max 2 iterations)
□ COMMIT: git commit with descriptive message
□ CHECKPOINT: /checkpoint <plan> <task> completed
□ STOP: Session pauses here - user runs /execute-plan to continue
```

<search_first_guardrail>
Before implementing ANY task, search the codebase first:
1. Search for existing implementations of the feature/fix
2. Grep for related function names, class names, patterns
3. ONLY proceed if confirmed the functionality doesn't already exist

"Don't assume not implemented" - this prevents duplicate work and conflicts.
</search_first_guardrail>

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

## Step 4: Mandatory Session Stop After Each Task

**One task per session.** This ensures:
- Fresh context for each task (prevents degradation)
- Code review cannot be skipped
- Checkpoint state is always current
- Failures are isolated to single tasks

**After EACH task completion, you MUST:**

1. Save checkpoint: `/checkpoint <plan> <task> completed`
2. Run code review: `/pr-review-toolkit:review-pr staged`
3. Commit the work
4. **STOP and output:**

```
✓ Task [N] complete.

Session will pause for context refresh.

To continue with next task:
  /execute-plan <plan-name>

Progress: [N]/[total] tasks complete
Next task: [N+1] - [title]
```

**Do NOT continue to the next task in the same session.**

This is not optional. Each task = one session = one commit = one review cycle.

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

When committing, pre-commit hooks may fail. **User consent is REQUIRED for any bypass.**

<hook_bypass_rule>
NEVER use `--no-verify` without explicit user consent via AskUserQuestion.
This is a HARD BLOCK - no exceptions, no assumptions, no autonomous decisions.
</hook_bypass_rule>

**When bypass might be appropriate (still requires user consent):**
- Pre-existing lint errors not introduced by your changes
- Formatting issues from automated tools (prettier, eslint --fix)
- Hook runs checks unrelated to your changes (e.g., unmodified files)

**When bypass is NEVER acceptable:**
- New lint errors in files you modified
- Type errors in your code
- Failing tests
- Security vulnerabilities flagged by hooks

**Decision flow:**
```
Hook failed → Check if error is in files YOU modified
  ├── Yes → Fix the issue, do NOT bypass
  └── No → STOP and ask user using AskUserQuestion:
           "Pre-commit hook failed on pre-existing issues.
            - Error: [summary]
            - Files affected: [list]
            Bypass with --no-verify?"
            Options: [Yes, bypass] [No, fix first]
```

**If user explicitly approves bypass:**
```bash
git commit --no-verify -m "your message"
```

**Document ALL bypasses in checkpoint:**
```yaml
session_log:
  - event: hook_bypassed
    reason: "Pre-existing lint errors in unmodified files"
    user_consent: true
    timestamp: "..."
```

**If user declines bypass:** Stop execution, report the issue, do not proceed.

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

- **During execution:** `/checkpoint` called automatically
- **After tasks:** Rotation check, may recommend session clear
- **After completion:** Suggest `/pr-review-toolkit:review-pr` for final review
- **To resume:** Run `/execute-plan <plan>` again - checkpoint state is loaded automatically
