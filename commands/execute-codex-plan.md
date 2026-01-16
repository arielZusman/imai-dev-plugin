---
description: Orchestrate Codex CLI plan execution with checkpoint, review, and commit management
argument-hint: <plan-name> [task-number]
arguments:
  plan_name:
    description: Name of plan folder in docs/plans/codex/ (e.g., "2026-01-16-feature-name")
    required: true
  task_number:
    description: Optional specific task to execute (defaults to checkpoint position)
    required: false
---

# Execute Codex Plan Command

Orchestrates execution of Codex CLI plans with automated checkpoint management, code review, and git commits.

## Overview

This command implements the Codex-Claude automation workflow:

1. **Claude Code handles:** Checkpoints, verification, code review, git commits
2. **Codex CLI handles:** Code implementation only
3. **One task per session:** Ensures fresh context and mandatory code review

## Workflow

### Step 1: Load Plan & Checkpoint

#### 1.1 Locate Plan

Plan is located at: `docs/plans/codex/<plan-name>/`

```bash
# Plan structure:
docs/plans/codex/YYYY-MM-DD-feature-name/
├── intro.md
├── task-0.md (optional)
├── task-1.md
├── task-2.md
├── ...
└── checkpoint.md
```

#### 1.2 Read Plan Content

1. Read `intro.md`:
   - Extract task count from Task Index
   - Note any special execution instructions
   - Load shared code context (for reference)

2. Read `checkpoint.md` (if exists):
   - Get `execution.current_task`
   - Get `execution.status` (in_progress, completed, blocked)
   - Get `git.last_commit` and `git.working_tree_clean`
   - Load recent session_log entries

3. Determine starting task:
   - If `$ARGUMENTS.task_number` provided → use that
   - If checkpoint exists AND status != completed → resume current_task
   - If checkpoint.status == completed → next_task = current_task + 1
   - If no checkpoint → start from Task 0 (or Task 1 if no Task 0)

#### 1.4 Load Task File

Read current task file: `docs/plans/codex/<plan-name>/task-<N>.md`

Extract:
- Task title
- Complexity
- Context Requirements
- Steps
- Verification commands
- Failure modes

**Output summary:**
```
📋 Executing Codex Plan: [plan-name]

Position: Task [N] of [total]
Title: [task title]
Complexity: [🟢/🟡/🔴]

Plan: ✅ Loaded
Checkpoint: ✅ Loaded (resuming) OR ⚠️ None (fresh start)
```

### Step 2: Pre-Flight Verification

Run baseline checks to ensure clean starting state:

```bash
# 1. Verify baseline build (establishes what should pass)
npm run build

# 2. Verify baseline tests
npm run test

# 3. Check git status (conditional - see below)
git status --short
```

**Conditional git status:**
- If checkpoint exists AND `git.working_tree_clean == true` AND `git.last_commit == current HEAD`:
  → Skip git status (already verified)
- Otherwise:
  → Run `git status` to detect uncommitted changes

**Expected:** Clean working tree OR only untracked files

**If pre-flight checks fail:**
```
❌ Pre-flight verification failed

[Error details from build/test/git]

Please fix baseline issues before executing plan.

Common fixes:
- npm install (if dependencies issue)
- git stash (if uncommitted changes)
- Fix failing tests

Then retry: /execute-codex-plan <plan-name>
```

Stop execution.

**If checks pass:**
```
✅ Pre-flight verification passed
- Build: ✅ Passing
- Tests: ✅ [X] passed, [Y] skipped
- Git: ✅ Clean working tree
```

### Step 3: Task Execution (ONE Task Per Session)

Execute the current task through the following steps:

#### 3.1 Create Checkpoint (Task Started)

```bash
/checkpoint docs/plans/codex/<plan-name>/intro.md <task-number> started
```

This updates checkpoint.md with:
- `execution.current_task = N`
- `execution.phase = implement`
- `execution.status = in_progress`
- Session log entry: `task_started`
- Current git state

#### 3.2 Invoke Codex CLI

**Correct invocation (per official Codex CLI docs):**

```bash
cd <project-root>

# Read task file content
TASK_CONTENT=$(cat docs/plans/codex/<plan-name>/task-<N>.md)

# Execute with required flags
codex exec --full-auto "$(echo "$TASK_CONTENT")" 2>&1
```

**Required flags:**
- `--full-auto` - Allows Codex to edit files without confirmation prompts
- Output streams: Progress → stderr, final message → stdout
- Exit code: 0 = success, non-zero = failure

**Optional flags (for future enhancement):**
- `--json` - Machine-readable JSON Lines output for parsing
- `--sandbox danger-full-access` - If default sandbox is too restrictive
- `-o <path>` - Write final message to file

**Capture:**
- Combined stdout/stderr (2>&1 redirection)
- Exit code (0 = success, non-zero = failure)
- Modified files (from git status after execution)

**Expected Codex behavior:**
- Receives task content as prompt (inline code context included)
- Implements code changes per Steps section
- Writes modified files (with --full-auto permission)
- Returns final status message

**If Codex returns non-zero exit code:**
```
❌ Codex CLI execution failed

Exit code: [code]
Error output:
[stderr]

Retry count: [N]/2
```

- If retry count < 2:
  - Extract error details
  - Re-invoke Codex with error feedback
  - Go back to step 3.2
- If retry count >= 2:
  - Update checkpoint: status = blocked
  - Log error in session_log
  - Stop execution

**Output during execution:**
```
🤖 Invoking Codex CLI for Task [N]...

[Stream Codex output to user]

✅ Codex completed execution
Files modified: [list from git status]
```

#### 3.3 Verify Build

```bash
npm run build
```

**If build fails:**
```
❌ Build failed after Codex changes

Build errors:
[error output]

Retry count: [N]/2 (shared with Codex retry count)
```

- If retry count < 2:
  - Extract build errors
  - Re-invoke Codex with error feedback:
    ```
    Previous attempt failed with build errors:
    [errors]

    Please fix these issues and regenerate the code.
    ```
  - Go back to step 3.2
- If retry count >= 2:
  - Update checkpoint: status = blocked
  - Log blocker details in session_log
  - Stop execution

**If build passes:**
```
✅ Build passed
```

#### 3.4 Run Code Review

```bash
/pr-review-toolkit:review-pr staged
```

**pr-review-toolkit examines:**
- Staged changes (git diff --staged)
- Code quality issues
- Security vulnerabilities
- Style violations
- Logic errors

**Output:** List of issues with severity levels

#### 3.5 Handle Review Feedback

**If critical or high-severity issues found:**
```
⚠️ Code review found [N] critical/high severity issues:

[List of issues]

Retry count: [N]/2 (shared with Codex + build retry count)
```

- If retry count < 2:
  - Format issues as feedback for Codex:
    ```
    Code review found the following issues:

    Critical:
    - [issue 1]
    - [issue 2]

    High:
    - [issue 3]

    Please fix these issues and regenerate the code.
    ```
  - Re-invoke Codex CLI with review feedback
  - Go back to step 3.2
- If retry count >= 2:
  - Update checkpoint: status = blocked
  - Log blocker details:
    ```yaml
    session_log:
      - event: blocked
        reason: "Code review issues after max retries"
        issues: [list of unresolved issues]
        timestamp: "..."
    ```
  - Stop execution

**If only medium/low issues OR no issues:**
```
✅ Code review passed
[If medium/low issues: ⚠️ Minor issues noted but proceeding]

[List medium/low issues if any]
```

- Log issues (if any) but proceed to commit
- Continue to step 3.6

#### 3.6 Commit Changes

Generate commit message and commit:

```bash
git add .

# Determine commit type and scope
COMMIT_TYPE="feat"  # or fix, refactor, test, docs, chore (based on task)
TASK_TITLE="[task title from task file]"
TASK_NUM="[N]"
PLAN_NAME="[plan-name]"

# Get summary from task file or generate from changes
SUMMARY="[2-3 sentence summary of what changed]"

# Build verification status
BUILD_STATUS="✅ Passing"
TEST_STATUS="✅ Passing" # or "N/A if no tests"
REVIEW_STATUS="✅ Passed" # or "⚠️ Minor issues noted"

git commit -m "$(cat <<'EOF'
${COMMIT_TYPE}: ${TASK_TITLE}

${SUMMARY}

Build: ${BUILD_STATUS}
Tests: ${TEST_STATUS}
Review: ${REVIEW_STATUS}

Task: ${TASK_NUM} of ${TOTAL_TASKS} from ${PLAN_NAME}

Co-Authored-By: Codex AI <noreply@openai.com>
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

**If commit fails (pre-commit hook):**
- Do NOT automatically bypass
- Display error and ask user:
  ```
  ❌ Pre-commit hook failed

  [Error details]

  This may be due to:
  - Pre-existing issues in unmodified files
  - New issues introduced by changes

  How would you like to proceed?
  1. Fix issues and retry
  2. Bypass hook (--no-verify) if pre-existing issues
  3. Mark task as blocked
  ```

- Use AskUserQuestion for choice
- If user approves bypass: `git commit --no-verify -m "..."`
- Document bypass in checkpoint session_log

**If commit succeeds:**
```
✅ Changes committed
Commit: [commit hash]
```

#### 3.7 Update Checkpoint (Task Completed)

```bash
/checkpoint docs/plans/codex/<plan-name>/intro.md <task-number> completed
```

**Checkpoint command will:**
- Prompt for handoff notes (what next task needs to know)
- Update checkpoint.md:
  - `execution.status = completed`
  - Session log entry: `task_completed`
  - Capture final git state (new commit hash)
  - Generate continuation prompt for next session
- Enforce session stop

#### 3.8 Mandatory Session Stop

**Output to user:**
```
✅ Task [N] completed successfully!

📋 Summary:
- Task: [N] - [title]
- Files modified: [count]
- Build: ✅ Passing
- Tests: ✅ Passing
- Review: ✅ Passed (or ⚠️ Minor issues noted)
- Committed: [commit-hash]

⏸️  SESSION PAUSED (one-task-per-session policy)

This ensures:
- Fresh context for next task
- Mandatory code review enforcement
- Checkpoint accuracy
- Failure isolation

To continue to Task [N+1]:
  /execute-codex-plan <plan-name>

Or to see plan status:
  cat docs/plans/codex/<plan-name>/checkpoint.md
```

**DO NOT continue to next task** - stop here.

### Step 4: Final Verification (After All Tasks)

When `current_task == total_tasks` AND status == completed:

```bash
# Run full verification suite
npm run build
npm run test

# Check git state
git status
git log --oneline -n <total_tasks>  <!-- cspell:disable-line -->
```

**Output:**
```
🎉 All tasks completed!

📋 Plan: [plan-name]
Tasks: [total_tasks] completed

✅ Final verification:
- Build: ✅ Passing
- Tests: ✅ [X] passed, [Y] skipped
- Git: ✅ Clean working tree
- Commits: [total_tasks] commits created  <!-- cspell:disable-line -->

Next steps:
- Review changes: git diff main
- Create PR or merge to main branch
- Update plan status in intro.md
```

## Error Handling & Edge Cases

### Codex CLI Errors

**If Codex returns non-zero exit code:**

1. Capture error output
2. Check if error is recoverable:
   - Build errors → retry with feedback (max 2 attempts)
   - File not found → likely plan issue, mark blocked
   - Syntax errors → retry with feedback
   - Timeout → ask user to retry or increase timeout

3. If unrecoverable or max retries reached:
   - Update checkpoint: status = blocked
   - Log error in session_log
   - Prompt user for manual intervention:
     ```
     ❌ Task blocked after max retry attempts

     Error: [details]

     Manual intervention required:
     1. Review task file: docs/plans/codex/<plan>/task-<N>.md
     2. Fix issue manually or update task instructions
     3. Run /checkpoint <plan> <task> completed when resolved
     4. Continue with /execute-codex-plan <plan>
     ```

### Git Conflicts

**If working tree is dirty when starting:**

Use AskUserQuestion:
```
⚠️ Working tree has uncommitted changes

Git status:
[git status output]

How would you like to proceed?
```

Options:
1. Stash changes and continue (git stash)
2. Commit current changes first
3. Abort execution

Handle based on user choice.

### Checkpoint Corruption

**If checkpoint.md is malformed:**

1. Attempt to parse, extract what's possible
2. If completely unreadable:
   ```
   ⚠️ Checkpoint file corrupted or unreadable

   Location: docs/plans/codex/<plan>/checkpoint.md

   Options:
   1. Restart from Task 0 (create fresh checkpoint)
   2. Start from specific task number
   3. Abort execution
   ```
   - Use AskUserQuestion for choice
   - Create fresh checkpoint based on choice

### Missing Dependencies

**If Codex CLI not installed:**

Check for Codex CLI before execution:
```bash
which codex
```

**If not found, display error:**
```
❌ Codex CLI not installed

The /execute-codex-plan command requires the Codex CLI to be installed.

Installation:
  See https://developers.openai.com/codex for setup instructions

Authentication:
  Set CODEX_API_KEY environment variable or run 'codex login'

For CI/automation:
  export CODEX_API_KEY=<your-key>
  codex exec --full-auto "task"
```

Stop execution until Codex CLI is installed and authenticated.

### Build/Test Baseline Failures

**If pre-flight checks fail:**

```
❌ Baseline verification failed

Build: [status]
Tests: [status]
Git: [status]

Suggested fixes:
- npm install (if dependency issues)
- Check recent commits for breaking changes
- Review uncommitted changes
- Fix failing tests before proceeding

Execution stopped. Fix issues and retry:
  /execute-codex-plan <plan-name>
```

Stop execution - user must fix baseline first.

## Usage Examples

**Execute plan from current position (or start):**
```bash
/execute-codex-plan 2026-01-16-auth-feature
```

**Execute specific task:**
```bash
/execute-codex-plan 2026-01-16-auth-feature 5
```

**Resume after fixing blocked task:**
```bash
# After manual fix
/checkpoint docs/plans/codex/2026-01-16-auth-feature/intro.md 5 completed

# Continue to next task
/execute-codex-plan 2026-01-16-auth-feature
```

## Integration with Other Commands

**Workflow:**
1. Generate plan: `/generate-codex-plan <basic-plan>`
2. Execute tasks: `/execute-codex-plan <plan-name>` (repeat for each task)
3. Final review: `/pr-review-toolkit:review-pr all` (when all complete)

**Checkpoint integration:**
- `/checkpoint` called automatically during execution
- Manual checkpoint updates for error recovery
- Checkpoint drives task resumption

**Code review integration:**
- `/pr-review-toolkit:review-pr staged` runs after each task
- Review feedback passed to Codex for fixes
- Critical issues block task completion

## Design Rationale

### Why one task per session?

- **Fresh context:** Prevents context degradation across tasks
- **Mandatory review:** Cannot skip code review step
- **Checkpoint accuracy:** State always reflects current position
- **Failure isolation:** Issues contained to single task

### Why max 2 retry attempts?

- **Prevents infinite loops:** Clear failure mode
- **Matches existing patterns:** Consistent with execute-plan
- **Most issues resolve quickly:** 1-2 attempts usually sufficient
- **Encourages human intervention:** Complex issues need manual review

### Why direct Codex CLI invocation?

- **Simple and transparent:** Uses `codex exec --full-auto` for non-interactive execution
- **Easy to debug:** Clear stdout/stderr streams (2>&1 redirection)
- **Official automation support:** Codex has `--full-auto` and `--json` flags for scripting
- **Portable:** Works anywhere Codex CLI installed with CODEX_API_KEY set

## Notes

**Codex CLI task files are standalone:**
- Include inline code context (from intro.md)
- No dependencies on conversation history
- Can be executed independently

**Session management:**
- Each task = fresh session = one commit = one review cycle
- Checkpoint enables seamless resumption
- Clear handoff between sessions via continuation prompt

**Error recovery:**
- Retry loops handle transient issues
- Blocked status requires manual intervention
- All state preserved in checkpoint for debugging
