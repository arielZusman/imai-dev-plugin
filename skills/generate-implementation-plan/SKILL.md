---
name: generate-implementation-plan
description: |
  Converts basic plans into execution-ready documents with embedded context,
  verification steps, and mandatory code review workflow.

  Use after creating a plan, or when preparing plans for fresh sessions
  where inline context is essential for execution.
metadata:
  context: fork
  agent: Plan
---

# Generate Implementation Plan

**Announce:** "I'm using the generate-implementation-plan skill to enrich this plan for standalone execution. This will embed the mandatory execution workflow and per-task checklists."

## Quick Start

1. Identify plan from `~/.claude/plans/`
2. Read and gather context for referenced files
3. **Determine output format:**
   - **Multi-file (default):** If >= 3 tasks, use [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md) + [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
   - **Single-file:** If < 3 tasks OR user explicitly requests, use [TEMPLATE.md](assets/TEMPLATE.md)
4. Save to `docs/plans/`:
   - Multi-file: Create folder `YYYY-MM-DD-<feature-name>/` containing `intro.md` + `task-N.md` files
   - Single-file: `YYYY-MM-DD-<feature-name>.md`
5. Update `docs/plans/INDEX.md`

## Process

1. **Identify the plan** - Ask user which plan or use the most recent
2. **Read the plan** - Understand tasks and scope
3. **Determine format** - Count tasks; if >= 3 use multi-file, otherwise single-file (unless user overrides)
4. **Gather context** - Read relevant files mentioned in the plan
5. **Discover test files** - For each modified file, find related `.spec.ts` files and mock locations
6. **Compute checksums** - Calculate md5 checksums for files to be modified (first 8 chars)
7. **Gather dependency info** - For package upgrades, query peer dependencies and known issues
8. **Enrich** - Add self-contained header and inline code context
9. **Add verification** - How to confirm each task is complete
10. **Add skill recommendations** - Which skills to invoke per task
11. **Generate output:**

    **If multi-file format:**
    - Create intro file from [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md)
    - Create task files from [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
    - Build Task Index in intro with relative links to task files
    - Code snippets go in intro only; task files reference "See intro"

    **If single-file format:**
    - Apply [TEMPLATE.md](assets/TEMPLATE.md) (existing behavior)

12. **Save files:**
    - Multi-file: Create folder `docs/plans/YYYY-MM-DD-<feature>/`, write `intro.md` + all `task-N.md` files into it
    - Single-file: Write `docs/plans/YYYY-MM-DD-<feature>.md`
13. **Update INDEX** - Add/update entry in `docs/plans/INDEX.md` with format indicator

## Key Principles

The enriched plan is for a **fresh Claude session** with zero prior context.

## Multi-File Format: Content Split

When using multi-file format, content is split to minimize context overhead per session:

| Content | Location | Rationale |
|---------|----------|-----------|
| Project Context | Intro only | Shared metadata |
| Architecture | Intro only | Design decisions are global |
| Code Snippets | Intro only | Avoids duplicating 50-100 lines per task |
| Migration Patterns | Intro only | Reference patterns apply to all tasks |
| Execution Workflow | Intro only | Process is the same for all tasks |
| Execution Log | Intro only | Single source of truth for status |
| Orchestration Hints | Intro only | Parallel groups span tasks |
| Gotchas & Warnings | Intro only | Global concerns |
| Task Index | Intro only | Links to all task files |
| Task Details | Task file | Task-specific implementation |
| Context Requirements | Task file | What to read for THIS task |
| File Checksums | Task file | Task-specific files |
| Test File Discovery | Task file | Task-specific tests |
| Steps | Task file | Task-specific actions |
| Verification | Task file | Task-specific confirmation |
| Failure Modes | Task file | Task-specific recovery |
| Handoff Notes | Task file | Filled post-completion |
| Per-Task Checklist | Task file | Task-specific tracking |

**Task files reference shared content with:**
```markdown
> **Plan:** [[feature-name]](./intro.md)
> **See intro for:** Architecture, Code Context, Execution Workflow
```

### Migration Patterns (for upgrade plans)

For plans involving syntax migrations or API changes, include before/after examples in the "Relevant Code Context" section. See [MIGRATION-PATTERNS.md](references/MIGRATION-PATTERNS.md) for common patterns.

Example format:
```markdown
### Migration Pattern: *ngIf to @if

**Before:**
```html
<div *ngIf="user">{{ user.name }}</div>
```

**After:**
```html
@if (user) {
  <div>{{ user.name }}</div>
}
```

**Notes:** Remove ng-template wrappers when using @else
```

- Include all context inline (no memory of your conversation)
- Be explicit about file locations and patterns
- Reference relevant code snippets directly in the plan
- Intent for business logic, exact content for templates/config
- Verification criteria for every task
- Skill recommendations for specialized workflows

## Context Optimization (Reduces Startup Tool Calls)

Pre-compute context that would otherwise require tool calls during execution:

### Test File Discovery
For each modified file, include:
- Related `.spec.ts` file path
- Mock setup location (line numbers)
- Required mock changes based on interface modifications

### File Checksums
For each file to be modified:
- Compute `md5 -q <file> | cut -c1-8`
- Include "lines to modify" range
- Enables skip-if-already-done detection

### Dependency Information
For package upgrade tasks:
- Query `npm info <pkg> peerDependencies`
- Document known compatibility issues
- Include breaking changes from changelogs

## Skill Recommendations

For each task, recommend relevant skills that should be invoked. Check all available skills including:
- Built-in skills (e.g., from superpowers plugin)
- Project-specific skills (in `.claude/skills/`)
- Third-party plugin skills

| Task Type | Recommended Skill | When to Use |
|-----------|------------------|-------------|
| Writing tests first | `superpowers:test-driven-development` | TDD workflow, test before implementation |
| Debugging issues | `superpowers:systematic-debugging` | Bug fixes, unexpected behavior |
| Multiple independent tasks | `superpowers:dispatching-parallel-agents` | 2+ tasks can run in parallel |
| Planning implementation | `superpowers:writing-plans` | Multi-step feature planning |
| Code review | `superpowers:requesting-code-review` | After completing tasks |
| Session boundaries | `handoff-summary` | Before rotating sessions |
| Session health | `session-management` | Context degradation suspected |

**Per-task format:**
```markdown
**Recommended skill:** `superpowers:test-driven-development`
  - Invoke BEFORE writing implementation code
  - Ensures tests are written first
```

**Selection criteria:**
- Match task type to skill purpose
- Use `—` if no skill provides clear benefit
- Include invocation timing (before/during/after implementation)

## Complexity Assignment

Every task MUST be assigned a complexity rating during enrichment:

**🟢 Simple** (< 50 lines, single focus):
- Single file modifications < 50 lines
- Config changes, simple refactors
- Clear input → output transformations
- Examples: Add environment variable, update constant, simple helper function

**🟡 Moderate** (50-200 lines, some complexity):
- Multiple files modified
- Business logic implementation
- Requires understanding existing patterns
- Examples: Add API endpoint, implement validation logic, refactor component

**🔴 Complex** (> 200 lines, architectural):
- Architectural changes
- Cross-cutting concerns
- Requires deep domain knowledge
- Examples: Migrate authentication system, redesign data layer, add new service

**Dispatch implications:**
- 🟢: Direct execution or focused-task-executor (if also < 30 lines, single file)
- 🟡: Sub-agent recommended for context isolation
- 🔴: Sub-agent required

**Who assigns:** Plan enricher during enrichment process (not the original planner)

**Validation:** Plan validator checks all tasks have complexity assigned

## Agent Recommendations

When a specialist agent would benefit a task, recommend it. Common patterns:

| Task Type | Agent Type | When to Use |
|-----------|------------|-------------|
| Test writing | `general-purpose` | TDD with fresh context, invoke skill first |
| Complex implementation | `general-purpose` | Context isolation benefit |
| Code exploration | `Explore` | Understanding unfamiliar code |
| Architecture decisions | `Plan` | Design decisions, trade-offs |

**Selection criteria:**
- Match task domain to agent specialty
- Use `—` if no agent provides clear benefit
- Consider project-specific agents if available

## Codex Delegation (Optional)

For users who prefer OpenAI Codex for code generation tasks, add `Dispatch: codex` to the task.

**Prerequisites:**
- Run `/setup-codex` to configure Codex MCP server
- Codex CLI installed and authenticated

**When to recommend Codex:**
- User explicitly requests Codex for tasks
- Code generation heavy tasks (large implementations)
- When user has stated preference for Codex style

**Format:**
```markdown
**Dispatch:** codex
```

**Important:** Codex delegation only handles implementation. Claude Code still manages:
- Checkpoint creation (before/after task)
- Code review (`/pr-review-toolkit:review-pr`)
- Verification and commit

## Critical: Execution Workflow Embedding

The enriched plan MUST include these elements to ensure code reviews are not skipped:

1. **Execution Workflow section** - Placed BEFORE tasks, not buried at the end
2. **Per-task checklist** - Each task ends with a checklist including the review step

This workflow mandates:
- Code review for EVERY task (`/pr-review-toolkit:review-pr`)
- Status updates in Execution Log
- Fix-and-retry cycle for critical issues (max 2 cycles)
- Session handoff protocol

Without these, executors will skip reviews because the instructions are too far from where they're working.

## References

**Templates:**
- **Multi-file intro template**: [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md) (default for >= 3 tasks)
- **Multi-file task template**: [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
- **Single-file template**: [TEMPLATE.md](assets/TEMPLATE.md) (legacy, < 3 tasks)

**Guides:**
- **Enrichment checklist**: [ENRICHMENT-CHECKLIST.md](references/ENRICHMENT-CHECKLIST.md)
- **Failure modes examples**: [FAILURE-MODES-EXAMPLES.md](references/FAILURE-MODES-EXAMPLES.md)
- **Migration patterns**: [MIGRATION-PATTERNS.md](references/MIGRATION-PATTERNS.md)
- **Execution guide**: [EXECUTION-GUIDE.md](references/EXECUTION-GUIDE.md)

## Execution Handoff

After saving the enriched plan, offer:

**For multi-file format:**
> **Plan enriched and saved:**
> - Intro: `docs/plans/<feature>.intro.md`
> - Tasks: `docs/plans/<feature>.task-0.md` through `task-N.md`
>
> Ready to execute?
> 1. **Execute now** - Start working through tasks in this session
> 2. **New session** - Open fresh session, run `/execute-plan <feature-name>`

**For single-file format:**
> **Plan enriched and saved to `docs/plans/<filename>.md`. Ready to execute?**
>
> 1. **Execute now** - Start working through tasks in this session
> 2. **New session** - Open fresh session with just the plan file

## INDEX.md Maintenance

After saving an enriched plan, update `docs/plans/INDEX.md`:

1. Read current INDEX.md
2. Check if plan already exists in table
3. If new: Add row to "Active Plans" with:
   - Link to plan file (intro file for multi-file, main file for single-file)
   - **Format** column: `multi` or `single`
   - Created date (from filename)
   - Task count (count task files for multi-file, or `### Task N:` headers for single-file)
   - Progress: 0/N
   - Branch (from plan's Git Context)
   - Status: ⏳ Not Started
4. If existing: Update progress and status based on Execution Log
5. Write updated INDEX.md

**INDEX.md format with multi-file support:**
```markdown
| Plan | Format | Created | Tasks | Progress | Branch | Status |
|------|--------|---------|-------|----------|--------|--------|
| [Feature A](./2026-01-14-feature-a/intro.md) | multi | 2026-01-14 | 8 | 3/8 | feature/a | 🔄 |
| [Feature B](./2026-01-13-feature-b.md) | single | 2026-01-13 | 2 | 0/2 | feature/b | ⏳ |
```

When a plan reaches 100% completion:
- Move from "Active Plans" to "Completed Plans"
- Add completion date
