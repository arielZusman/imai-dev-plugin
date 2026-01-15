# Implementation Plan Template

Copy this template when enriching a plan. Fill in all bracketed sections.

---

```markdown
# [Feature Name] Implementation Plan

<execution_context>
Execute this plan task-by-task. All context needed is included below.
Implement changes directly. Do not suggest or ask for confirmation unless blocked.
</execution_context>

## Project Context

**Repository:** [repo name and path]
**Service:** [which service/module this affects]
**Branch:** [feature branch name]
**Original plan:** `~/.claude/plans/[plan-filename].md`

## Fresh Session Entry Point

When resuming this plan in a new session, read in this order:
1. **Git state:** `git log --oneline -5` and `git status`
2. **This plan's Execution Log** (bottom of document)
3. **Checkpoint file:** `docs/plans/.state/<slug>.checkpoint.md` (if exists)
4. **Current task's Context Requirements**

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
3. Run `/execute-plan <plan-name>` in fresh session

### Critical vs Non-Critical

- **Critical (blocks commit):** Security vulnerabilities, breaking bugs, failing tests, logic errors
- **Non-critical (note and proceed):** Style suggestions, minor refactoring, documentation gaps

### If Blocked

Mark status as `⛔ Blocked` with notes. Resolve before continuing to next task.

### Final Review

After all tasks complete, run `/pr-review-toolkit:review-pr all` and update the "Final Review" row.
</workflow>

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

### Task 0: Verify Prerequisites

**Why this task matters:**
- **Business:** Prevents wasted time from missing dependencies or environment issues
- **Technical:** Validates environment matches plan assumptions

**Complexity:** 🟢 Simple
**Estimated time:** ~5 min
**Dispatch:** direct

**Steps:**
1. [Command from Prerequisites: e.g., `node --version` → expect v18.19.0+]
2. [Command: e.g., `npm --version` → expect 9+]
3. [Project-specific check: e.g., `docker ps | grep postgres`]

**Verify:** All commands return expected output.

**Checklist:**
- [ ] All prerequisites verified
- [ ] Ready to proceed with Task 1

---

### Task 1: [Title]

**Why this task matters:**
- **Business:** [How this serves the user/product goal]
- **Technical:** [Why this approach/order is correct]

**Complexity:** 🟢 Simple | 🟡 Moderate | 🔴 Complex
**Estimated time:** ~15 min | ~30 min | ~1 hr
**Depends on:** None | Task N, Task M
**Parallel group:** A | — (sequential)
**Dispatch:** direct | sub-agent ([agent-name]) | codex
**Recommended skill:** `[skill-name]` | — (none)
  - [When to invoke: BEFORE/DURING/AFTER implementation]
  - [What the skill provides]

**Context Requirements:**
Read these files before implementing. Do not speculate about code you haven't opened.

- **Required** (must re-read before starting):
  - `exact/path/to/file.ts` - sections: [functionName, className]
    - **Verify:** [What state/pattern to confirm before proceeding]
  - `exact/path/to/types.ts` - all
    - **Verify:** [Expected types/interfaces present]
- **Reference** (consult if needed):
  - `docs/architecture.md` - sections: [relevant section]

**Files:**
- Modify: `exact/path/to/file.ts:45-60`
- Create: `exact/path/to/new-file.ts`

**Steps:**
1. [Intent + location: "Add email validation function to validators.ts:45"]
2. [Command: "Run tests to verify"]

**Verify:** [How to confirm this task is complete]
**Verification Method:** Manual test | Automated test | MCP tool | Build check

**Review scope:** [Files modified in this task]

**Failure Modes:**
- **If [symptom]:** Likely cause is [X]. Fix by [Y].
- **Rollback:** `git checkout HEAD -- [files modified]` or [specific undo steps]

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
| Task 2 | sub-agent (general-purpose) | TDD test writing |
| Task 3 | sub-agent (general-purpose) | Complex implementation |
| Task 4 | codex | User prefers Codex for implementation |
| Task N | direct | Config/trivial change |

**Dispatch guidelines:**
- `direct`: Simple edits, config changes, < 50 lines modified
- `sub-agent`: TDD tests, complex implementation, code review
- `codex`: User preference for OpenAI Codex (requires `/setup-codex` first)
- Only delegate to sub-agents/codex when the task clearly benefits from a separate context

---

## Verification Checklist

- [ ] All tests pass: `npm run test`
- [ ] Build succeeds: `npm run build`
- [ ] [Feature-specific verification]

## Gotchas & Warnings

- **Do not hardcode values** to make tests pass. Implement the actual logic that solves the problem generally.
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
