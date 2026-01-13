# Implementation Plan Template (Codex)

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
**Original plan:** `~/.codex/plans/[plan-filename].md`

## Goal

[One sentence describing what this builds]

## Architecture

[2-3 sentences about the approach and key design decisions]

---

## Execution Workflow

<mandatory_workflow>
Follow this workflow for EVERY task. Do not skip steps.

### Per-Task Cycle

1. **Checkpoint Start:** `/prompts:checkpoint PLAN_PATH=<plan-path> TASK_NUMBER=<n> STATUS=started`
2. **Context Load:** Re-read files listed in task's "Context Requirements"
3. **Implement:** Complete the task steps
4. **Verify:** Run the task's verification step
5. **Review:** Review with `git diff --staged` and `git status`
6. **Fix if needed:** Address critical issues (max 2 cycles per task)
7. **Commit:** After review passes, commit with descriptive message
8. **Checkpoint Complete:** `/prompts:checkpoint PLAN_PATH=<plan-path> TASK_NUMBER=<n> STATUS=completed`
9. **STOP:** One task per session. Run `/prompts:execute-plan PLAN=<plan-name>` to continue.

### Critical vs Non-Critical

- **Critical (blocks commit):** Security vulnerabilities, breaking bugs, failing tests, logic errors
- **Non-critical (note and proceed):** Style suggestions, minor refactoring, documentation gaps

### If Blocked

Mark status as `blocked` with notes. Do NOT proceed to next task.
</mandatory_workflow>

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

**Why:** [Business reason this task matters]

**Complexity:** Simple | Moderate | Complex
**Depends on:** None | Task N, Task M
**Parallel group:** None (sequential)
**Dispatch:** direct
**Recommended skill:** `[skill-name]` | none
  - [When to invoke]
  - [What the skill provides]

**Context Requirements:**
> ALWAYS read these files before implementing. Do not speculate about code you haven't opened.

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
**Verification Method:** Manual test | Automated test | Build check

**Review scope:** [Files modified in this task]

**Failure Modes:**
- **If [symptom]:** Likely cause is [X]. Fix by [Y].

**Handoff Notes:** (fill after completion)
- [What the next task needs to know about this implementation]

**Checklist:**
- [ ] `/prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=started`
- [ ] Context requirements re-read
- [ ] Implementation complete
- [ ] Verification passed
- [ ] Review: `git diff --staged` and `git status`
- [ ] Committed
- [ ] `/prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=completed`

---

## Task Execution Order

**Sequential tasks:** Execute in order, respecting "Depends on" field.

---

## Verification Checklist

- [ ] All tests pass: [project-specific command]
- [ ] Build succeeds: [project-specific command]
- [ ] [Feature-specific verification]

## Gotchas & Warnings

- **Do not hardcode values** to make tests pass. Implement the actual logic that solves the problem generally.
- [Any non-obvious issues to watch for]
- [Dependencies or order-of-operations concerns]

---

## Execution Log

| Task | Status | Review | Committed | Notes |
|------|--------|--------|-----------|-------|
| Task 1 | Pending | | | |
| Task 2 | Pending | | | |
| Final Review | - | Pending | | |

**Status Legend:** Pending | In Progress | Done | Blocked
**Review Legend:** Pending | Fixing | Passed
```
