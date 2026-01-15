# Multi-File Plan: Intro Template

Copy this template when generating a multi-file enriched plan. This is the intro file that contains shared context. Individual task files use [TASK-TEMPLATE.md](./TASK-TEMPLATE.md).

**Directory structure:**
```
docs/plans/YYYY-MM-DD-<feature-name>/
├── intro.md          # This file
├── task-0.md         # Prerequisites
├── task-1.md         # First task
├── task-N.md         # Additional tasks
└── checkpoint.md     # Execution state (created during execution)
```

---

```markdown
# [Feature Name] Implementation Plan

<execution_context>
Execute this plan task-by-task. Each task is in a separate file.
Implement changes directly. Do not suggest or ask for confirmation unless blocked.
</execution_context>

## Plan Metadata

| Field | Value |
|-------|-------|
| Format | multi-file |
| Created | YYYY-MM-DD |
| Task Count | N |
| Branch | [feature branch name] |
| Original Plan | `~/.claude/plans/[plan-filename].md` |

## Project Context

**Repository:** [repo name and path]
**Service:** [which service/module this affects]
**Target Directory:** [absolute path to working directory]

## Fresh Session Entry Point

When resuming this plan in a new session:

1. **Read this intro file** (you are here)
2. **Check git state:** `git log --oneline -5` and `git status`
3. **Check checkpoint:** `./checkpoint.md` (if exists)
4. **Find current task:** Look at Task Index below or checkpoint for next pending task
5. **Load task file:** Read only the task file for current task (e.g., `./task-3.md`)

Then continue from the first non-completed task.

## Goal

[One sentence describing what this builds]

## Architecture

[Write 2-3 sentences in prose describing the approach, key design decisions, and how components interact. Avoid bullet points here—use narrative flow to explain why this architecture was chosen and how the pieces fit together.]

---

## Execution Workflow

<workflow>
Follow this workflow for each task:

### Per-Task Cycle

1. **Load Task File:** Read the task file for current task number
2. **Checkpoint Start:** `/checkpoint <plan-path> <task-number> started`
3. **Context Load:** Re-read files listed in task's "Context Requirements"
4. **Implement:** Complete the task steps
5. **Verify:** Run the task's verification step
6. **Review:** Run `/pr-review-toolkit:review-pr` with aspects matching changes:
   | Changes | Command |
   |---------|---------|
   | Code only | `code` |
   | + Error handling | `code errors` |
   | + Types | `code types` |
   | + Tests | `code tests` |
7. **Fix if needed:** Address critical issues (max 2 cycles per task)
8. **Commit:** After review passes, commit with descriptive message
9. **Checkpoint Complete:** `/checkpoint <plan-path> <task-number> completed`
10. **Session Stop:** Do NOT continue to next task in same session

**To continue:** Run `/execute-plan <plan-name>` in fresh session. The checkpoint state will load automatically.

**Why stop after each task:**
- Fresh context prevents degradation
- Code review cannot be skipped
- Failures are isolated to single tasks

### Critical vs Non-Critical

- **Critical (blocks commit):** Security vulnerabilities, breaking bugs, failing tests, logic errors
- **Non-critical (note and proceed):** Style suggestions, minor refactoring, documentation gaps

### If Blocked

Mark status as `⛔ Blocked` with notes in the Execution Log. Resolve before continuing to next task.

### Final Review

After all tasks complete, run `/pr-review-toolkit:review-pr all` and update the "Final Review" row.

### Commit Message Format

```
<type>: <task-title> (Task N)

<optional detailed description>

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Types:** feat, fix, refactor, test, docs, chore

**Examples:**
- `feat: add email validation (Task 3)`
- `fix: resolve auth token expiry bug (Task 5)`
- `test: add unit tests for user service (Task 2)`
</workflow>

---

## Relevant Code Context

[Include code snippets here that are referenced by multiple tasks. Task files will reference "See intro for code context".]

### [File 1 - brief description]
`path/to/file.ts:10-45`
```typescript
[relevant code snippet]
```

### [File 2 - brief description]
`path/to/other.ts:100-120`
```typescript
[relevant code snippet]
```

---

## Migration Patterns

[For upgrade/migration plans only. Include before/after syntax examples that apply across tasks.]

### Pattern: [Name]
**Before:**
```[lang]
[old syntax]
```

**After:**
```[lang]
[new syntax]
```

**Notes:** [Any caveats or edge cases]

---

## Task Index

[List all task files with links. This is the single source of truth for task list.]

| Task | Title | File | Complexity | Dispatch | Status |
|------|-------|------|------------|----------|--------|
| 0 | Verify Prerequisites | [task-0](./task-0.md) | 🟢 | direct | ⏳ |
| 1 | [Title] | [task-1](./task-1.md) | 🟡 | sub-agent | ⏳ |
| 2 | [Title] | [task-2](./task-2.md) | 🟢 | direct | ⏳ |
| ... | ... | ... | ... | ... | ... |

---

## Task Execution Order

**Sequential tasks:** Execute in order, respecting "Depends on" field in each task file.

**Parallel groups:** Tasks with same group letter can run simultaneously:
- Group A: Tasks 1, 2 (independent, run in parallel)
- Group B: Tasks 4, 5 (depend on Group A, run in parallel after A completes)
- Sequential (—): Task 3 (has dependencies, must wait)

**Parallelization rules:**
- Only parallelize tasks with NO shared file modifications
- All dependencies must be complete before starting
- Use Task tool to spawn parallel agents when beneficial

---

## Orchestration Hints

> For `/execute-plan` command - guides dispatch decisions

**Parallel groups:**
- Group A: [task_1, task_2] - Independent, can run simultaneously
- Group B: [task_4, task_5] - Depend on Group A, parallel after A completes

**Dispatch decisions:**
| Task | Dispatch | Rationale |
|------|----------|-----------|
| Task 0 | direct | Prerequisites, trivial |
| Task 1 | direct | Simple, < 50 lines |
| Task 2 | sub-agent (general-purpose) | TDD test writing |
| Task 3 | sub-agent (general-purpose) | Complex implementation |
| Task N | codex | User prefers Codex for implementation |

**Dispatch guidelines:**
- `direct`: Simple edits, config changes, < 50 lines modified
- `sub-agent`: TDD tests, complex implementation, code review
- `codex`: User preference for OpenAI Codex (requires `/setup-codex` first)
- Only delegate to sub-agents/codex when the task clearly benefits from a separate context

---

## Verification Checklist

[Plan-level verification after all tasks complete]

- [ ] All tests pass: `npm run test`
- [ ] Build succeeds: `npm run build`
- [ ] [Feature-specific verification]

## Gotchas & Warnings

- **Do not hardcode values** to make tests pass. Implement the actual logic that solves the problem generally.
- [Any non-obvious issues to watch for]
- [Dependencies or order-of-operations concerns]

---

## Execution Log

[Single source of truth for task status. Update this table as tasks complete.]

| Task | Status | Review | Committed | Notes |
|------|--------|--------|-----------|-------|
| Task 0 | ⏳ Pending | | | |
| Task 1 | ⏳ Pending | | | |
| Task 2 | ⏳ Pending | | | |
| Final Review | - | ⏳ | | |

**Status Legend:** ⏳ Pending | 🔄 In Progress | ✅ Done | ⛔ Blocked
**Review Legend:** ⏳ Pending | 🔄 Fixing | ✅ Passed
```
