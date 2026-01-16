# Codex Execution Plans Index

This directory contains enriched implementation plans optimized for Codex CLI execution.

## About Codex Plans

**Division of Labor:**
- **Codex CLI:** Handles all code changes (reading files, editing code, creating files)
- **Claude Code:** Handles checkpoints, verification, code review, commits, tests

**Format:**
- All plans use multi-file format to support automation
- Each plan has its own directory: `YYYY-MM-DD-<feature-name>/`
- Directory contains: `intro.md` + `task-N.md` files

**Execution:**
- Each task file can be passed to Codex CLI independently
- Enables scripting automation to iterate through tasks
- See intro.md in each plan for execution workflow

---

## Active Plans

| Plan | Created | Tasks | Progress | Branch | Status |
|------|---------|-------|----------|--------|--------|
| _(none yet)_ | - | - | - | - | - |

**Status Legend:**
- ⏳ Not Started
- 🔄 In Progress
- ✅ Completed
- ⛔ Blocked

---

## Completed Plans

| Plan | Created | Tasks | Completed | Branch |
|------|---------|-------|-----------|--------|
| _(none yet)_ | - | - | - | - |

---

## How to Create a Codex Plan

1. Create a basic plan outline
2. Run `/generate-codex-plan <plan-path>` to enrich it
3. The enriched plan will be saved to this directory
4. Each task can be executed independently with Codex CLI

## How to Execute a Codex Plan

### Recommended: Automated Execution

Use the `/execute-codex-plan` command for fully automated orchestration:

```bash
/execute-codex-plan YYYY-MM-DD-feature-name
```

**What it does:**
1. Loads plan and checkpoint state (resumes from last position)
2. Runs pre-flight verification (build, tests, git status)
3. Creates checkpoint (task started)
4. Invokes Codex CLI for code implementation
5. Verifies build passes
6. Runs code review (`/pr-review-toolkit:review-pr staged`)
7. Handles retry loop for failures (max 2 attempts)
8. Commits changes with proper message
9. Updates checkpoint (task completed)
10. **Stops session** (one task per session for fresh context)

**To continue:** Run the same command again. It automatically resumes from the next task.

**Benefits:**
- Automated checkpoint management
- Mandatory code review (cannot be skipped)
- Consistent commit messages
- Automatic retry handling
- Session isolation per task

### Alternative: Manual Execution

For manual control:

1. Read `<plan-name>/intro.md`
2. For each task:
   - Load task file (e.g., `task-1.md`)
   - Pass to Codex CLI: `TASK_CONTENT=$(cat task-N.md) && codex exec --full-auto "$(echo "$TASK_CONTENT")"`
   - Use Claude Code for verification and review
   - Create checkpoint and commit manually

See the task's intro.md for detailed execution workflow.
