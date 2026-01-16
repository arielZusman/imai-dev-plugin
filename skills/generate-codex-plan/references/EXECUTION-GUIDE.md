# Execution Guide (Codex Plans)

How to execute an enriched Codex implementation plan.

> **Note:** The execution workflow is embedded directly in each enriched plan under the "Execution Workflow" section. This guide provides supplementary information.

## Division of Labor

**Codex CLI handles:**
- Reading files
- Editing code
- Creating new files
- Following implementation steps

**Claude Code handles:**
- Checkpoints (`/checkpoint`)
- Running tests and build
- Code review (`/pr-review-toolkit:review-pr`)
- Git commits
- Verification

## Resuming a Plan

Any session can resume an enriched Codex plan by:
1. Reading the intro file from `docs/plans/codex/<plan-name>/intro.md`
2. Scanning the Execution Log for current position (first non-✅ task)
3. Loading the current task file (e.g., `task-3.md`)
4. Following the embedded Execution Workflow

## Per-Task Cycle

### For Codex CLI:
1. **Load task file** - Read `task-N.md` for current task
2. **Read context** - Read all files listed in "Context Requirements"
3. **Implement steps** - Execute each step in the "Steps (for Codex CLI)" section
   - Read files as needed
   - Edit existing files
   - Create new files
4. **Confirm completion** - Verify all Codex checklist items done

### For Claude Code (after Codex):
1. **Checkpoint start** - `/checkpoint <plan-path> <task-number> started`
2. **Verify** - Run commands in "Verification (for Claude Code)" section
   ```bash
   npm run build
   npm run test -- [pattern]
   ```
3. **Review** - Run code review with appropriate aspects:
   ```bash
   /pr-review-toolkit:review-pr staged code
   # Add aspects: errors, types, tests (as needed)
   ```
4. **Fix if needed** - Address critical issues (max 2 cycles)
   - Use Codex CLI for code fixes
   - Re-run verification and review
5. **Commit** - After review passes:
   ```
   <type>: <task-title> (Task N)

   Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
   Co-Authored-By: OpenAI Codex <noreply@openai.com>
   ```
6. **Checkpoint complete** - `/checkpoint <plan-path> <task-number> completed`
7. **STOP** - Do not continue to next task in same session

## Parallel Execution

Tasks in the same parallel group can run simultaneously:

1. **Check parallel group** - Look at "Parallel group" field in task
2. **Verify independence** - Ensure no shared file modifications
3. **Run Codex instances** - Start separate Codex CLI instance for each task
4. **Verify independently** - Each task gets its own verification and review
5. **Commit independently** - Each task commits separately

**Example:**
```
Group A: Task 1, Task 2 (independent)
- Run: codex exec --full-auto "$(cat task-1.md)" (instance 1)
- Run: codex exec --full-auto "$(cat task-2.md)" (instance 2)
- Verify and commit each independently
```

## Automation Support

### Recommended: Use /execute-codex-plan Command

The `/execute-codex-plan` command provides fully automated orchestration:

```bash
# Execute one task per session
/execute-codex-plan YYYY-MM-DD-feature-name

# Automatically handles:
# - Checkpoint management
# - Codex CLI invocation
# - Build verification
# - Code review
# - Retry loops (max 2 attempts)
# - Git commits
# - Session isolation
```

**Benefits:**
- Mandatory code review enforcement
- Consistent error handling
- Automatic checkpoint state management
- One task per session for fresh context

### Alternative: Custom Automation Script

For custom automation workflows, the multi-file format enables scripting:

```bash
#!/bin/bash
PLAN_DIR="docs/plans/codex/2026-01-15-feature-x"

for task_file in $PLAN_DIR/task-*.md; do
  task_num=$(basename $task_file .md | sed 's/task-//')

  echo "Starting Task $task_num with Codex..."
  TASK_CONTENT=$(cat $task_file)
  codex exec --full-auto "$(echo "$TASK_CONTENT")" 2>&1

  echo "Verifying with Claude Code..."
  claude checkpoint $PLAN_DIR $task_num started
  npm run build && npm run test

  echo "Reviewing..."
  claude review staged code

  echo "Committing..."
  git commit -m "feat: implement task $task_num"

  claude checkpoint $PLAN_DIR $task_num completed
done
```

## Critical vs Non-Critical Issues

**Critical (blocks commit):**
- Security vulnerabilities
- Breaking bugs
- Failing tests
- Logic errors

**Non-critical (note and proceed):**
- Style suggestions
- Minor refactoring
- Documentation gaps

## If Blocked

If a task cannot be completed:
1. Mark status as `⛔ Blocked` in Execution Log
2. Add notes describing the blocker
3. DO NOT continue to next task
4. Resolve blocker before proceeding

## Final Review

After all tasks complete:
1. Run comprehensive review: `/pr-review-toolkit:review-pr all`
2. Update "Final Review" row in Execution Log
3. Verify plan-level checklist items
4. Create pull request if on feature branch

## Tips for Effective Execution

1. **One task at a time** - Complete fully before moving to next
2. **Fresh sessions** - Start new session for each task to prevent context degradation
3. **Use Codex for code** - Don't manually edit code that Codex should handle
4. **Don't skip review** - Code review is mandatory for every task
5. **Update logs** - Keep Execution Log current as single source of truth
