# Enrichment Checklist

Verify each item before saving the enriched plan.

## Multi-File Format Validation

When using multi-file format (>= 3 tasks), verify these additional items:

### Intro File (`<plan-folder>/intro.md`)
- [ ] **Format field** - `Format: multi-file` in Plan Metadata table
- [ ] **Task Index table** - Links to all task files with relative paths (`./task-N.md`)
- [ ] **Code snippets** - All snippets in "Relevant Code Context" section (NOT in task files)
- [ ] **Execution Log** - Status table with row for each task (single source of truth)
- [ ] **Migration patterns** - In intro only (if applicable)

### Task Files (`<plan-folder>/task-N.md`)
- [ ] **Reference header** - Links back to intro file (`./intro.md`) with "See intro for" note
- [ ] **No duplicate code snippets** - Task files reference intro for code context
- [ ] **Self-contained execution** - All steps, verification, failure modes included
- [ ] **Checksums and test files** - Task-specific, not shared

### Directory Structure
- [ ] **Plan folder:** `docs/plans/YYYY-MM-DD-<feature-name>/`
- [ ] **Intro file:** `intro.md` in plan folder
- [ ] **Task files:** `task-0.md`, `task-1.md`, etc. in plan folder
- [ ] **Checkpoint:** `checkpoint.md` in plan folder (created during execution)
- [ ] **Task 0** - Prerequisites verification task exists

---

## Required Content (Both Formats)

### Plan Structure
- [ ] **Project context** - Repo, service, branch
- [ ] **Fresh Session Entry Point** - Reading order for session resume (git state, execution log, checkpoint, context)
- [ ] **Goal** - One sentence summary
- [ ] **Architecture** - Design approach in prose (2-3 sentences, avoid bullets)
- [ ] **Execution Workflow section** - At TOP (after Architecture, BEFORE tasks) with per-task cycle
- [ ] **Relevant code context** - Key snippets the executing Claude needs
- [ ] **Migration patterns** - For upgrade plans, include before/after examples

### Per-Task Requirements
- [ ] **Task 0 for prerequisites** - Explicit verification before Task 1 (see below)
- [ ] **Why field with Business + Technical** - Both contexts explained, not just "needed for feature"
- [ ] **Time estimate** - ~15 min (🟢), ~30 min (🟡), ~1 hr (🔴)
- [ ] **Exact file paths** - With line numbers for modifications
- [ ] **Context Requirements with Verify notes** - REQUIRED vs REFERENCE files, plus what to check in each (see below)
- [ ] **Test file discovery** - Related .spec.ts files with mock locations (see below)
- [ ] **File checksums** - For modified files, enables skip-if-already-done (see below)
- [ ] **Clear intent or exact content** - Intent for logic, exact content for templates/config
- [ ] **Step granularity** - Each step is ONE action (2-5 min), not a bundle of actions
- [ ] **Commands with expected output** - What to run, what specific output to expect
- [ ] **Verification per task** - How to confirm completion
- [ ] **Failure modes with Rollback** - "If X happens, likely cause is Y, fix by Z" plus rollback command (see below)
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

## Why Field Guidelines

Each task should explain why it matters, not just what to do.

**Format:**
```markdown
**Why this task matters:**
- **Business:** [How this serves the user/product goal]
- **Technical:** [Why this approach/order is correct]
```

**Guidelines:**
- **Business context:** What user-facing or operational problem does this solve?
- **Technical context:** Why this approach vs. alternatives? Why this order?
- **Avoid:** Generic phrases like "needed for the feature" - be specific

**Good example:**
```markdown
**Why this task matters:**
- **Business:** Users currently see stale data after refresh; this ensures real-time sync
- **Technical:** Must run before Task 3 because the cache invalidation depends on the new event handler
```

---

## Time Estimates

Add per-task time estimate to help users plan sessions.

**Format:**
```markdown
**Estimated time:** ~15 min | ~30 min | ~1 hr
```

**Guidelines based on complexity:**
- 🟢 Simple (< 20 lines): ~15 min
- 🟡 Moderate (20-50 lines): ~30 min
- 🔴 Complex (50+ lines): ~1 hr

These help users decide if a task fits remaining session time.

---

## Task 0: Prerequisite Verification

Every plan should include Task 0 to verify prerequisites before starting.

**Format:**
```markdown
### Task 0: Verify Prerequisites

**Complexity:** 🟢 Simple
**Estimated time:** ~5 min
**Dispatch:** direct

**Steps:**
1. `node --version` → expect v18.19.0+
2. `npm --version` → expect 9+
3. [Project-specific check]

**Verify:** All commands return expected output.

**Checklist:**
- [ ] All prerequisites verified
- [ ] Ready to proceed with Task 1
```

---

## Content Format

Balance prose and lists appropriately.

**Use prose for:**
- Architecture section (2-3 flowing sentences)
- Why sections (explanatory context)
- Design rationale

**Use lists for:**
- Steps (discrete actions)
- Files (enumerated items)
- Checklists (completion tracking)

**Use natural language:**
- Prefer direct imperatives over MUST/CRITICAL markers
- Save emphasis for genuine security or data-loss scenarios
- Example: "Complete the review step before committing" vs "You MUST NOT skip the review step"

---

## Context Requirements

Each task should specify files the executor must re-read before starting, plus what to verify in each.

**Format:**
```markdown
**Context Requirements:**
- **Required** (must re-read before starting):
  - `src/services/auth.service.ts` - sections: validateToken, refreshToken
    - **Verify:** Still has old API signature that we're changing
  - `src/types/auth.types.ts` - all
    - **Verify:** AuthResponse interface exists with expected fields
- **Reference** (consult if needed):
  - `docs/auth-flow.md` - sections: Token Lifecycle
```

**Guidelines:**
- **Required**: Files that will be modified or whose behavior must be understood
- **Reference**: Background context, architecture docs, related but not modified files
- **Verify notes**: What state the file should be in, what pattern to look for
- Include section hints when file is large (don't re-read entire 500-line file)
- For TDD tasks, required includes test file from previous task

**Why Verify notes matter:**
- Prevents reading a file without knowing what you're checking
- Catches unexpected changes from previous tasks or other sessions
- Confirms assumptions before implementation begins

---

## Test File Discovery

Each task that modifies source files should identify related test files.

**Format:**
```markdown
**Affected Test Files:**
- `src/services/auth.service.spec.ts`
  - Mock location: lines 45-67
  - Required changes: Add mockQuerySegmentId, update AuthResponse mock
- `src/components/login/login.component.spec.ts`
  - Mock location: lines 12-30
  - Required changes: Update dependency injection mocks
```

**Discovery process:**
1. For each file to be modified (`foo.ts`), search for `foo.spec.ts` in same directory
2. Also search for test files that import the modified file
3. Identify mock setup locations within each test file
4. Document what mock changes are needed based on interface changes

**Benefits:**
- Eliminates 2-4 Glob/Grep calls per task to find test files
- Executor doesn't guess where mocks are located
- Pre-computed mock changes prevent trial-and-error

---

## File Checksums

Include checksums for files to be modified. Enables skip-if-already-done logic.

**Format:**
```markdown
**Target file:** `/path/to/file.ts`
**Checksum before edit:** `a1b2c3d4` (first 8 chars of md5)
**Lines to modify:** 45-89
```

**Compute during enrichment:**
```bash
md5 -q /path/to/file.ts | cut -c1-8
```

**Benefits:**
- Executor can verify file hasn't changed unexpectedly
- If resuming a task, checksum mismatch indicates task already completed
- Skip "did I already do this?" verification reads

---

## Dependency Information

For tasks involving package upgrades, pre-compute dependency chains.

**Format:**
```markdown
**Package Upgrade Context:**
- Package: `@angular/core@19`
- Peer requirements:
  - zone.js >= 0.15.0
  - rxjs >= 7.8.0
- Known issues:
  - Requires `--legacy-peer-deps` for @hakimio packages
  - TypeScript must be 5.4+
- Breaking changes:
  - `ComponentRef.changeDetectorRef` renamed to `cdr`
  - Standalone components now default
```

**Discovery during enrichment:**
```bash
npm info @angular/core@19 peerDependencies
npm info @angular/core@19 peerDependenciesMeta
```

**Benefits:**
- Eliminates 2-3 Bash calls per package to discover peer deps
- Known issues prevent repeated trial-and-error with --legacy-peer-deps
- Breaking changes prevent implementation guesswork

---

## Failure Modes

Document common failure patterns and rollback commands to help executor self-diagnose and recover.

**Format:**
```markdown
**Failure Modes:**
- **If build fails with "Cannot find module":** Likely missing import. Check imports at top of file.
- **If tests fail with "timeout":** Async operation not awaited. Check for missing `await`.
- **If [symptom]:** Likely cause is [X]. Fix by [Y].
- **Rollback:** `git checkout HEAD -- [files modified]`
```

**Common patterns to document:**
- Import/export mismatches
- Type mismatches (especially after refactoring)
- Missing dependencies (npm install needed)
- Order-of-operations issues
- Environment variable requirements
- Database migration dependencies

**Rollback patterns by task type:**
- **File changes:** `git checkout HEAD -- path/to/files`
- **Package updates:** Revert package.json + `npm ci`
- **Migrations:** `npm run migration:revert`
- **Multi-file feature:** `git stash` or `git reset HEAD~1`

See [FAILURE-MODES-EXAMPLES.md](FAILURE-MODES-EXAMPLES.md) for comprehensive examples.

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

**Mandatory session stop reminder in workflow:**
```markdown
### Session Stop

**After EACH task:** Do NOT continue to next task in same session.

**To continue:** Run `/execute-plan <plan-name>` in fresh session. Checkpoint state loads automatically.

**Why this matters:**
- Fresh context prevents degradation
- Code review cannot be skipped
- Failures are isolated to single tasks
```
