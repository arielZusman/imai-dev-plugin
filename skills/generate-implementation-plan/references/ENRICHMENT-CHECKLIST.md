# Enrichment Checklist

Verify each item before saving the enriched plan.

## Required Content

### Plan Structure
- [ ] **Project context** - Repo, service, branch
- [ ] **Goal** - One sentence summary
- [ ] **Architecture** - Design approach
- [ ] **Execution Workflow section** - At TOP (after Architecture, BEFORE tasks) with mandatory per-task cycle
- [ ] **Relevant code context** - Key snippets the executing Claude needs

### Per-Task Requirements
- [ ] **Exact file paths** - With line numbers for modifications
- [ ] **Context Requirements** - REQUIRED vs REFERENCE files per task (see below)
- [ ] **Clear intent or exact content** - Intent for logic, exact content for templates/config
- [ ] **Step granularity** - Each step is ONE action (2-5 min), not a bundle of actions
- [ ] **Commands with expected output** - What to run, what specific output to expect
- [ ] **Verification per task** - How to confirm completion
- [ ] **Failure modes** - "If X happens, likely cause is Y, fix by Z" (see below)
- [ ] **Handoff notes placeholder** - Space to document what next task needs to know
- [ ] **Per-task checklist** - Including checkpoint start/complete calls

### Task Coordination
- [ ] **Task dependencies** - "Depends on" field for each task
- [ ] **Parallel groups** - Independent tasks grouped for parallel execution
- [ ] **Complexity indicators** - Each task marked 🟢/🟡/🔴
- [ ] **Dispatch hints** - `direct` vs `sub-agent` per task (see below)
- [ ] **Agent recommendations** - Per-task agent when beneficial (use `—` if none helps)

### Plan-Level
- [ ] **Orchestration Hints section** - Parallel groups and dispatch decisions table
- [ ] **Final checklist** - Tests, build, feature verification
- [ ] **Gotchas** - Non-obvious issues
- [ ] **Review protocol** - Per-task review scope, final review step
- [ ] **Execution Log** - Table with row per task, all marked ⏳ Pending

## Content Guidelines

**Include exact content when:**
- Translation keys (exact strings matter)
- HTML templates (structure must be exact)
- Config values (specific values required)
- Simple boilerplate (pattern is exact copy)

**Use intent + location when:**
- Business logic (Claude writes better code seeing actual context)
- Complex functions (needs adaptation to current code)
- Conditional logic (context-dependent)

**Rule of thumb:** Include exact content if the executing Claude needs the *exact characters*. Use intent if Claude can derive it from context.

---

## Context Requirements

Each task should specify files the executor MUST re-read before starting.

**Format:**
```markdown
**Context Requirements:**
- **Required** (must re-read before starting):
  - `src/services/auth.service.ts` - sections: validateToken, refreshToken
  - `src/types/auth.types.ts` - all
- **Reference** (consult if needed):
  - `docs/auth-flow.md` - sections: Token Lifecycle
```

**Guidelines:**
- **Required**: Files that will be modified or whose behavior must be understood
- **Reference**: Background context, architecture docs, related but not modified files
- Include section hints when file is large (don't re-read entire 500-line file)
- For TDD tasks, required includes test file from previous task

---

## Failure Modes

Document common failure patterns to help executor self-diagnose.

**Format:**
```markdown
**Failure Modes:**
- **If build fails with "Cannot find module":** Likely missing import. Check imports at top of file.
- **If tests fail with "timeout":** Async operation not awaited. Check for missing `await`.
- **If [symptom]:** Likely cause is [X]. Fix by [Y].
```

**Common patterns to document:**
- Import/export mismatches
- Type mismatches (especially after refactoring)
- Missing dependencies (npm install needed)
- Order-of-operations issues
- Environment variable requirements
- Database migration dependencies

**When to include:**
- Complex tasks (🟡/🔴 complexity)
- Tasks that modify shared code
- Tasks with non-obvious prerequisites
- Tasks where you've seen failures in similar work

---

## Dispatch Hints

Guide the `/execute-plan` command on whether to run task directly or dispatch to sub-agent.

**Format per task:**
```markdown
**Dispatch:** direct | sub-agent ([agent-name])
```

**Dispatch decision guidelines:**

| Criteria | Dispatch | Rationale |
|----------|----------|-----------|
| < 50 lines changed | `direct` | Low overhead wins |
| Config/env changes | `direct` | Trivial, no isolation benefit |
| TDD test writing | `sub-agent (general-purpose)` | Fresh context prevents impl bias |
| Complex implementation | `sub-agent (general-purpose)` | Isolation prevents context rot |
| Code review | `sub-agent (pr-review-toolkit:code-reviewer)` | Already isolated workflow |
| Parallel-eligible | `sub-agent` | Required for parallelism |

**Orchestration Hints section format:**
```markdown
## Orchestration Hints

**Parallel groups:**
- Group A: [task_1, task_2] - Independent, can run simultaneously
- Group B: [task_4, task_5] - Depend on Group A, parallel after A

**Dispatch decisions:**
| Task | Dispatch | Rationale |
|------|----------|-----------|
| 1 | direct | Interface definitions, < 20 lines |
| 2 | sub-agent (general-purpose) | TDD red phase |
| 3 | sub-agent (general-purpose) | Complex algorithm |
| 4 | direct | Config update |
```

---

## Checkpoint Integration

Plans should reference checkpoint workflow in execution steps.

**Per-task checklist must include:**
```markdown
**Checklist:**
- [ ] `/checkpoint <plan> <task> started`
- [ ] Context requirements re-read
- [ ] Implementation complete
- [ ] Verification passed
- [ ] Review: `/pr-review-toolkit:review-pr [aspects]`
- [ ] Committed
- [ ] `/checkpoint <plan> <task> completed`
```

**Rotation heuristic reminder in workflow:**
```markdown
### Rotation Heuristic

After 4 tasks or 30 minutes, consider rotating session:
1. Run `/checkpoint` to save state
2. Clear session
3. Run `/resume-plan <plan-path>` in fresh session
```
