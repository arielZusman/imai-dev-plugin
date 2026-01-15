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

Use **absolute paths** in all commands. Claude Code knows the working directory from environment context - no need to verify with `pwd`.

**If plan specifies a different target directory:**
- Read from plan's "Target Directory" field
- Use absolute paths for all file operations
- Only `cd` if a command requires relative paths and cannot use absolute

Running commands in the wrong directory corrupts the wrong `package.json`.

---

## Overview

This command implements **lean orchestration**:
- Simple tasks execute directly in main session (low overhead)
- Complex tasks dispatch to specialist sub-agents (context isolation)
- Parallel-eligible tasks can run concurrently via multiple sub-agents
- Checkpoint updates after each task (MANDATORY - do not skip)
- Rotation recommendations based on task count/time

## Step 1: Detect Format and Load Plan

### 1.0 Detect Plan Format

Check for multi-file format first (subdirectory), fall back to single-file:

**Multi-file detection (subdirectory):**
```
docs/plans/*$ARGUMENTS.plan*/intro.md
```

**If folder with intro.md exists:** Multi-file format
- Plan folder: `docs/plans/YYYY-MM-DD-feature/`
- Intro: `docs/plans/YYYY-MM-DD-feature/intro.md`
- Tasks: `docs/plans/YYYY-MM-DD-feature/task-N.md`
- Checkpoint: `docs/plans/YYYY-MM-DD-feature/checkpoint.md`
- Slug: folder name (e.g., `YYYY-MM-DD-feature`)

**If folder does NOT exist:** Single-file format (legacy)
- Plan: `docs/plans/*$ARGUMENTS.plan*.md`
- Slug: `YYYY-MM-DD-feature` (remove `.md`)
- Checkpoint: `docs/plans/.state/YYYY-MM-DD-feature.checkpoint.md`

### 1.1 Construct Paths

**Checkpoint path differs by format:**

**Multi-file (subdirectory):**
```
Plan folder:     docs/plans/2026-01-14-feature/
Checkpoint path: docs/plans/2026-01-14-feature/checkpoint.md
```

**Single-file (legacy):**
```
Slug:            2026-01-14-feature
Checkpoint path: docs/plans/.state/2026-01-14-feature.checkpoint.md
```

Do NOT Glob for the checkpoint file - the path is deterministic from format detection.

### 1.2 Load Plan Content

**If multi-file format:**
1. Read intro file for:
   - Task Index (list of task file paths)
   - Execution Log (current state)
   - Code Context (shared snippets)
   - Orchestration Hints (parallel groups, dispatch decisions)
2. Do NOT read all task files - only load current task file (in Step 1.5)

**If single-file format:**
- Read entire plan file (existing behavior)
- Parse task list with metadata from within the file

### 1.3 Parse Plan State

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

4. Confirm session policy:
   ```
   ℹ️ Session stop after each task (automatic rotation)
   Next task will run in fresh session via `/execute-plan <plan-name>`
   ```

## Step 1.5: Load Task-Specific Context

<investigate_before_answering>
Read the actual Context Requirements from the checkpoint or plan. Do not guess file paths or assume what context is needed.
</investigate_before_answering>

### For Multi-File Format

1. **Get current task number** from checkpoint or Execution Log
2. **Construct task file path:**
   ```
   <plan-folder>/task-<N>.md
   Example: docs/plans/2026-01-14-feature/task-3.md
   ```
3. **Read the task file** - contains all task-specific details
4. **For code snippets:** Task file says "See intro for code context"
   - Read referenced snippets from intro's "Relevant Code Context" section
5. **Read Context Requirements** listed in the task file

### For Single-File Format

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

**From plan (fallback):**

Read the task's "Context Requirements" field from within the plan file.

### Both Formats

**Re-read only required files**, not the entire codebase.

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

## Step 2: Pre-Flight Verification (Optimized)

Skip redundant checks to reduce startup overhead. The previous session already verified build/tests on completion.

**Conditional checks based on checkpoint state:**

```bash
# 1. Verify test baseline (establishes what should pass)
npm run test

# 2. Check git status ONLY IF:
#    - No checkpoint exists, OR
#    - Checkpoint is stale (last_commit doesn't match HEAD), OR
#    - Checkpoint shows working_tree_clean: false
#    OTHERWISE: Skip - checkpoint already confirms clean state
git status  # Conditional
```

**Skip these (redundant):**
- ~~`pwd`~~ - Working directory is already known from Claude Code's environment context
- ~~`npm run build`~~ - Build was verified at end of previous session. If checkpoint shows task completed, build already passed.
- ~~`cd <plan-target-directory>`~~ - Use absolute paths instead. Only cd if plan explicitly requires different directory.

**Conditional git status logic:**
```
If checkpoint exists AND checkpoint.git.working_tree_clean == true AND checkpoint.git.last_commit == HEAD:
  → Skip git status (already verified)
Else:
  → Run git status (detect uncommitted changes from crashed sessions or external modifications)
```

**Output a status summary:**
```
Pre-flight check:
- Tests: ✅ X passed, Y skipped
- Git: ✅ clean (from checkpoint) OR ✅ verified clean
```

**If pre-flight fails:**
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
| `Dispatch: focused-task-executor` in task | Dispatch to focused-task-executor | Lightweight model for trivial edits |
| `Dispatch: codex` in task | Delegate to Codex via MCP | User preference for Codex |
| Complexity 🟢 + single file + < 30 lines | Dispatch to focused-task-executor | Cost-efficient lightweight model |
| Complexity 🟢 + < 50 lines | Execute directly | Low overhead wins |
| Complexity 🟡/🔴 | Dispatch to sub-agent | Context isolation benefit |
| TDD test task | Dispatch to test-writer | Fresh context for test design |
| Implementation after tests | Dispatch to implementer | Isolation from test context |

### 3.2 Execute Task

<checklist_tracking>
Use TodoWrite to track checklist progress for EVERY task. This is not optional.

**Why this matters:**
- Users cannot see your internal state - TodoWrite gives them visibility
- Skipping tracking leads to skipped steps - the todo list enforces completeness
- Each completed item proves the step was done, not just planned

**How to track:**
1. Before starting a task, create ALL checklist items as todos (status: pending)
2. Mark each item `in_progress` as you start it
3. Mark each item `completed` IMMEDIATELY after finishing - do not batch completions

Do NOT proceed to the next task until all checklist items show completed.
</checklist_tracking>

### Task Checklist (create as TodoWrite todos)

| Step | content | activeForm |
|------|---------|------------|
| 1 | Save checkpoint: task started | Saving task started checkpoint |
| 2 | Search for existing implementation | Searching for existing implementation |
| 3 | Execute task steps | Implementing task |
| 4 | Run task verification command | Running verification |
| 5 | Run build (npm run build) | Building project |
| 6 | Run code review (/pr-review-toolkit:review-pr staged) | Running code review |
| 7 | Fix critical issues (max 2 iterations) | Fixing review issues |
| 8 | Commit changes | Committing changes |
| 9 | Save checkpoint: task completed | Saving task completed checkpoint |

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
2. Read task's Context Requirements (required files)
3. Execute task steps as written in plan (use absolute paths)
4. Run task's Verify step
5. Run `/pr-review-toolkit:review-pr staged` (see mandatory_code_review above)
6. Fix critical issues (max 2 iterations)
7. Commit with descriptive message
8. Run `/checkpoint <plan> <task> completed`
9. Provide handoff notes when prompted

**After completing all steps:** Verify your TodoWrite list shows all 9 items as `completed` before announcing task completion.

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

**If dispatching to focused-task-executor (Haiku):**

Use for 🟢 complexity, single-file, <30 line changes. This agent uses the Haiku model for cost efficiency.

1. Run `/checkpoint <plan> <task> started`
2. Build task prompt with FULL file content (Haiku needs explicit context)
3. Dispatch via Task tool:
   ```
   Task tool:
   - subagent_type: general-purpose
   - model: haiku
   - prompt: [built prompt - must include file content, not just paths]
   - description: "Execute Task N: [title]"
   ```
4. Wait for agent completion
5. Verify results:
   - Check file was modified as expected
   - Run verification step
   - Run code review
6. If verification fails or agent reports "blocked": do NOT retry with Haiku - escalate to direct execution or sub-agent
7. Commit changes
8. Run `/checkpoint <plan> <task> completed`

**Focused-task-executor prompt requirements:**
- Include FULL content of target file (not just path)
- Specify exact change needed (no ambiguity)
- Include verification command
- Tell agent to report back if uncertain (no improvising)

**After sub-agent completes:** Update your TodoWrite list to mark relevant items as `completed`. Verify all items complete before proceeding.

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

### 3.4 Codex Delegation

When dispatch decision is `codex`, delegate the implementation to OpenAI's Codex via MCP.

**Prerequisites:**
- Codex MCP server configured (run `/setup-codex` if not)
- If MCP tool `mcp__codex__codex` is not available, warn user and offer alternatives

**Delegation flow:**

1. Run `/checkpoint <plan> <task> started`

2. Build delegation prompt using template from `commands/references/CODEX-DELEGATION-TEMPLATE.md`:
   - Populate all fields (task title, context, steps, etc.)
   - **Inline all required files** - Codex is stateless, needs full file contents
   - Include relevant code patterns from codebase

3. Determine sandbox mode:
   - `workspace-write` for implementation tasks (creates/modifies files)
   - `read-only` for analysis/review only

4. Call MCP tool:
   ```
   mcp__codex__codex
   - prompt: [built delegation prompt]
   - sandbox: [workspace-write or read-only]
   ```

5. Parse Codex response:
   - Extract file blocks (look for `### File:` headers)
   - Parse code content between triple backticks
   - Validate paths match expected files

6. Apply changes:
   - Use Write tool for each modified file
   - Preserve any files Codex didn't modify

7. Continue to verification (same as direct execution):
   - Run task's Verify step
   - Run `/pr-review-toolkit:review-pr staged`
   - Fix critical issues (max 2 iterations)
   - Commit with descriptive message
   - Run `/checkpoint <plan> <task> completed`

**If Codex delegation fails:**

Do NOT silently fall back. Ask user via AskUserQuestion:

```
Codex delegation failed.
Error: [error details]

How would you like to proceed?
```

Options:
- **Retry Codex** - Try delegation again (useful for transient errors)
- **Fall back to Claude** - Execute directly in main session
- **Mark blocked** - Stop execution, require manual intervention

Log outcome in checkpoint:
```yaml
session_log:
  - event: codex_delegation_failed
    error: "[error details]"
    resolution: "[retry|fallback|blocked]"
    timestamp: "..."
```

**Example Codex task in plan:**

```markdown
### Task 5: Implement validation service

**Complexity:** 🟡 Moderate
**Dispatch:** codex

**Context Requirements:**
- Required: `src/services/validation.service.ts`
- Required: `src/types/validation.types.ts`
- Reference: `src/services/auth.service.ts` (pattern example)

**Steps:**
1. Create ValidationService class
2. Add validateEmail method with RFC 5322 regex
3. Add validatePassword method (min 8 chars, 1 number, 1 special)
4. Export from services/index.ts

**Verify:**
- `npm run build` passes
- `npm run test -- validation` passes
```

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

**Task: Add new import statement to service file (🟢 Simple)**
```
Dispatch: focused-task-executor
Rationale: Single file, < 5 lines, mechanical change - ideal for Haiku
```

**Task: Rename function and update JSDoc (🟢 Simple)**
```
Dispatch: focused-task-executor
Rationale: Single file, < 20 lines, clear transformation - cost-efficient with Haiku
```

**Task: Implement data transformation pipeline (🟡 Moderate)**
```
Dispatch: codex
Rationale: User prefers Codex for code generation heavy tasks
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
