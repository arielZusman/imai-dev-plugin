# Multi-File Plan: Task Template

Copy this template when generating individual task files for a multi-file plan. The intro file uses [INTRO-TEMPLATE.md](./INTRO-TEMPLATE.md).

**Location:**
- Task files: `docs/plans/YYYY-MM-DD-<feature-name>/task-N.md`
- N is 0-indexed (Task 0 = prerequisites verification)

---

```markdown
# Task [N]: [Title]

> **Plan:** [[feature-name]](./intro.md)
> **See intro for:** Architecture, Code Context, Execution Workflow, Migration Patterns

---

**Why this task matters:**
- **Business:** [How this serves the user/product goal]
- **Technical:** [Why this approach/order is correct]

**Complexity:** 🟢 Simple | 🟡 Moderate | 🔴 Complex
**Estimated time:** ~5 min | ~15 min | ~30 min | ~1 hr
**Depends on:** None | Task N, Task M
**Parallel group:** A | B | — (sequential)
**Dispatch:** direct | sub-agent ([agent-name]) | focused-task-executor | codex
**Recommended skill:** `[skill-name]` | — (none)
  - [When to invoke: BEFORE/DURING/AFTER implementation]
  - [What the skill provides]

---

## Context Requirements

Read these files before implementing. Do not speculate about code you haven't opened.

**See intro for code snippets** - this section lists what to read, snippets are in intro's "Relevant Code Context".

- **Required** (must re-read before starting):
  - `exact/path/to/file.ts` - sections: [functionName, className]
    - **Verify:** [What state/pattern to confirm before proceeding]
  - `exact/path/to/types.ts` - all
    - **Verify:** [Expected types/interfaces present]
- **Reference** (consult if needed):
  - `docs/architecture.md` - sections: [relevant section]

---

## Files

- Modify: `exact/path/to/file.ts:45-60`
- Create: `exact/path/to/new-file.ts`

**Checksums (before edit):**
- `exact/path/to/file.ts`: `a1b2c3d4`

**Affected Test Files:**
- `exact/path/to/file.spec.ts`
  - Mock location: lines 45-67
  - Required changes: [description of mock updates needed]

---

## Steps

1. [Intent + location: "Add email validation function to validators.ts:45"]
2. [Specific action or command]
3. ...

---

## Verification

**Verify:** [How to confirm this task is complete]
**Verification Method:** Manual test | Automated test | MCP tool | Build check

**Commands:**
```bash
npm run build
npm run test -- [relevant test pattern]
```

---

## Failure Modes

- **If [symptom]:** Likely cause is [X]. Fix by [Y].
- **If build fails with "Cannot find module":** Check imports at top of file.
- **Rollback:** `git checkout HEAD -- [files modified]`

---

## Handoff Notes

(Fill after completion)
- [What the next task needs to know about this implementation]
- [Any decisions made during implementation]
- [Unexpected findings or changes from plan]

---

## Checklist

- [ ] `/checkpoint <plan> <task> started`
- [ ] Context requirements re-read
- [ ] Implementation complete
- [ ] Verification passed
- [ ] Build passes: `npm run build`
- [ ] Review: `/pr-review-toolkit:review-pr staged [aspects]`
  - Aspects: `code` (always) + `errors` (if error handling) + `types` (if types modified) + `tests` (if tests added)
- [ ] Committed
- [ ] `/checkpoint <plan> <task> completed`
- [ ] **STOP** - Session pauses here
```

---

## Task 0 Variant (Prerequisites)

For Task 0, use this simplified format:

```markdown
# Task 0: Verify Prerequisites

> **Plan:** [[feature-name]](./intro.md)
> **See intro for:** Architecture, Project Context

---

**Why this task matters:**
- **Business:** Prevents wasted time from missing dependencies or environment issues
- **Technical:** Validates environment matches plan assumptions

**Complexity:** 🟢 Simple
**Estimated time:** ~5 min
**Dispatch:** direct

---

## Steps

1. `node --version` → expect v18.19.0+
2. `npm --version` → expect 9+
3. [Project-specific check: e.g., `docker ps | grep postgres`]
4. `cd [target-directory] && npm run build` → verify build passes
5. `npm run test` → verify tests pass

---

## Verification

**Verify:** All commands return expected output.
**Verification Method:** Manual verification

---

## Checklist

- [ ] All prerequisites verified
- [ ] Build passes
- [ ] Tests pass
- [ ] Ready to proceed with Task 1
```
