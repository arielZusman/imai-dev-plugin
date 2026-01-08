# Implementation Plan Template

Copy this template when enriching a plan. Fill in all bracketed sections.

---

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** Execute this plan task-by-task. All context needed is included below.

## Project Context

**Repository:** [repo name and path]
**Service:** [which service/module this affects]
**Branch:** [feature branch name]
**Original plan:** `~/.claude/plans/[plan-filename].md`

## Goal

[One sentence describing what this builds]

## Architecture

[2-3 sentences about the approach and key design decisions]

---

## Execution Workflow

> **MANDATORY:** Follow this workflow for EVERY task. Do not skip steps.

### Per-Task Cycle

1. **Checkpoint Start:** `/checkpoint <plan-path> <task-number> started`
2. **Context Load:** Re-read files listed in task's "Context Requirements"
3. **Implement:** Complete the task steps
4. **Verify:** Run the task's verification step
5. **Review:** Run `/pr-review-toolkit:review-pr` with aspects matching changes:
   | Changes | Command |
   |---------|---------|
   | Code only | `code` |
   | + Error handling | `code errors` |
   | + Types | `code types` |
   | + Tests | `code tests` |
6. **Fix if needed:** Address critical issues (max 2 cycles per task)
7. **Commit:** After review passes, commit with descriptive message
8. **Checkpoint Complete:** `/checkpoint <plan-path> <task-number> completed`
9. **Rotation Check:** If checkpoint warns about rotation, consider clearing session

### Rotation Heuristic

After 4 tasks or 30 minutes, consider rotating session:
1. Run `/checkpoint` to save state
2. Clear session
3. Run `/resume-plan <plan-path>` in fresh session

### Critical vs Non-Critical

- **Critical (blocks commit):** Security vulnerabilities, breaking bugs, failing tests, logic errors
- **Non-critical (note and proceed):** Style suggestions, minor refactoring, documentation gaps

### If Blocked

Mark status as `⛔ Blocked` with notes. Do NOT proceed to next task.

### Final Review

After ALL tasks: Run `/pr-review-toolkit:review-pr all` and update "Final Review" row.

---

## Relevant Code Context

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

## Tasks

[Copy tasks from original plan, enriched with exact file paths and verification steps]

### Task 1: [Title]

**Complexity:** 🟢 Simple | 🟡 Moderate | 🔴 Complex
**Depends on:** None | Task N, Task M
**Parallel group:** A | — (sequential)
**Dispatch:** direct | sub-agent ([agent-name])
**Recommended skill:** `[skill-name]` | — (none)
  - [When to invoke: BEFORE/DURING/AFTER implementation]
  - [What the skill provides]

**Context Requirements:**
- **Required** (must re-read before starting):
  - `exact/path/to/file.ts` - sections: [functionName, className]
  - `exact/path/to/types.ts` - all
- **Reference** (consult if needed):
  - `docs/architecture.md` - sections: [relevant section]

**Files:**
- Modify: `exact/path/to/file.ts:45-60`
- Create: `exact/path/to/new-file.ts`

**Steps:**
1. [Intent + location: "Add email validation function to validators.ts:45"]
2. [Command: "Run tests to verify"]

**Verify:** [How to confirm this task is complete]

**Review scope:** [Files modified in this task]

**Failure Modes:**
- **If [symptom]:** Likely cause is [X]. Fix by [Y].

**Handoff Notes:** (fill after completion)
- [What the next task needs to know about this implementation]

**Checklist:**
- [ ] `/checkpoint <plan> <task> started`
- [ ] Context requirements re-read
- [ ] Implementation complete
- [ ] Verification passed
- [ ] Review: `/pr-review-toolkit:review-pr [aspects]`
- [ ] Committed
- [ ] `/checkpoint <plan> <task> completed`

---

## Task Execution Order

**Sequential tasks:** Execute in order, respecting "Depends on" field.

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
| Task 1 | direct | Simple, < 50 lines |
| Task 2 | sub-agent (test-writer) | TDD test writing |
| Task 3 | sub-agent (implementer) | Complex implementation |
| Task N | direct | Config/trivial change |

**Dispatch guidelines:**
- `direct`: Simple edits, config changes, < 50 lines modified
- `sub-agent`: TDD tests, complex implementation, code review

---

## Verification Checklist

- [ ] All tests pass: `npm run test`
- [ ] Build succeeds: `npm run build`
- [ ] [Feature-specific verification]

## Gotchas & Warnings

- [Any non-obvious issues to watch for]
- [Dependencies or order-of-operations concerns]

---

## Execution Log

| Task | Status | Review | Committed | Notes |
|------|--------|--------|-----------|-------|
| Task 1 | ⏳ Pending | | | |
| Task 2 | ⏳ Pending | | | |
| Final Review | - | ⏳ | | |

**Status Legend:** ⏳ Pending | 🔄 In Progress | ✅ Done | ⛔ Blocked
**Review Legend:** ⏳ Pending | 🔄 Fixing | ✅ Passed
```
