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
3. **Output format:**
   - **Multi-file (always):** Use [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md) + [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
   - Create folder `YYYY-MM-DD-<feature-name>/` containing `intro.md` + `task-N.md` files
4. Update `docs/plans/INDEX.md`

## Process

1. **Identify the plan** - Ask user which plan or use the most recent
2. **Read the plan** - Understand tasks and scope
3. **Gather context** - Read relevant files mentioned in the plan
4. **Discover test files** - For each modified file, find related `.spec.ts` files and mock locations
5. **Compute checksums** - Calculate md5 checksums for files to be modified (first 8 chars)
6. **Gather dependency info** - For package upgrades, query peer dependencies and known issues
7. **Enrich** - Add self-contained header and inline code context
8. **Add verification** - How to confirm each task is complete
9. **Add skill recommendations** - Which skills to invoke per task
10. **Generate ALL output content at once with model selection:**

    **CRITICAL OPTIMIZATION**: Generate content for ALL files (intro + all tasks) in a single LLM response.
    Do NOT generate task files one-by-one.

    **Model selection for content generation:**

    **For each task:**

    a) **Complex sections (use Sonnet):**
       - Architecture analysis
       - Code context extraction
       - Custom failure modes
       - Dependency discovery and conflict resolution
       - Task complexity assignment

    b) **Boilerplate sections (use templates + Haiku):**
       - Load template from `assets/checklist-template.md`
       - Substitute variables: {{TASK_NAME}}, {{FILE_PATH}}, etc.
       - Use Haiku sub-agent for any custom text generation
       - Standard verification steps (use verification-template.md)
       - Common failure modes (use failure-modes-template.md)

    c) **Generation strategy:**
       - All Sonnet sections first (complex analysis)
       - All template substitutions second (simple replacement)
       - All Haiku sections last (parallel sub-agents for boilerplate)

    **Multi-file generation:**
    - In ONE response, generate:
      * Complete intro.md content from [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md)
      * Complete task-0.md content (if exists) from [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
      * Complete task-1.md content from [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
      * Complete task-2.md content from [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
      * ... (all task files)
    - Build Task Index in intro with relative links to task files
    - Code snippets go in intro only; task files reference "See intro"
    - Store each file's content in memory with clear labels:
      ```
      INTRO_CONTENT = "# Fix Slow..."
      TASK_0_CONTENT = "# Task 0:..."
      TASK_1_CONTENT = "# Task 1:..."
      ```

11. **Write ALL files in parallel:**

    **CRITICAL OPTIMIZATION**: Use a SINGLE message with MULTIPLE Write tool calls.
    Do NOT write files sequentially.

    First, create the directory if needed:
    ```markdown
    Creating directory and writing all plan files in parallel:
    ```

    Then make ONE tool call message containing ALL Write operations:
    - Write(docs/plans/YYYY-MM-DD-<feature>/intro.md, INTRO_CONTENT)
    - Write(docs/plans/YYYY-MM-DD-<feature>/task-0.md, TASK_0_CONTENT)
    - Write(docs/plans/YYYY-MM-DD-<feature>/task-1.md, TASK_1_CONTENT)
    - Write(docs/plans/YYYY-MM-DD-<feature>/task-2.md, TASK_2_CONTENT)
    - ... (all task files)

    **Why**: Parallel writes execute simultaneously. Sequential writes wait for each API roundtrip (~26-53s per file).
12. **Update INDEX** - Add/update entry in `docs/plans/INDEX.md`

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

### Code Structure Discovery (ast-grep Skill)

**For TypeScript/JavaScript/Python files referenced in the plan:**

**Before gathering context, invoke ast-grep skill for structure:**

1. **Invoke ast-grep skill:**
   ```markdown
   Skill("ast-grep", args="Find all functions in src/service.ts")
   ```

2. **ast-grep returns signatures** (200-500 tokens):
   - Function names with line numbers
   - Class definitions with methods
   - Import statements

3. **Analyze signatures** to identify relevant code (2-3 functions typically)

4. **Read only relevant sections:**
   ```markdown
   Read src/service.ts offset=<start_line> limit=<num_lines>
   ```

**When to use ast-grep:**
- ✅ Finding function signatures before reading implementation
- ✅ Locating class methods across large files
- ✅ Discovering imports/exports for dependency analysis
- ✅ Identifying interface definitions for type context
- ❌ Small files (< 100 lines) - just Read directly
- ❌ Config files, JSON, YAML - no AST benefit

**Result:** 50-80% token reduction for code context gathering

**Important:** Always invoke the ast-grep skill explicitly using `Skill("ast-grep", args="...")` - Claude Code doesn't auto-invoke skills.

### Test File Discovery (Batched)

**CRITICAL**: Use ONE Glob call to find all test files at once.

**Instead of:**
```bash
Glob for auth.service.spec.ts    # Call 1
Glob for user.service.spec.ts    # Call 2
Glob for api.service.spec.ts     # Call 3
```

**Do this:**
```bash
Glob for {auth,user,api}.service.spec.ts
# OR
Glob for **/*.service.spec.ts (then filter)
```

**Result**: One Glob call instead of N calls for N files.

For each modified file, include:
- Related `.spec.ts` file path
- Mock setup location (line numbers)
- Required mock changes based on interface modifications

### File Checksums (Batched)

**CRITICAL**: Compute checksums for ALL files in a SINGLE Bash call.

**Instead of:**
```bash
md5 -q /path/to/file1.ts | cut -c1-8  # Call 1
md5 -q /path/to/file2.ts | cut -c1-8  # Call 2
md5 -q /path/to/file3.ts | cut -c1-8  # Call 3
```

**Do this:**
```bash
for file in /path/to/file1.ts /path/to/file2.ts /path/to/file3.ts; do
  echo "$file: $(md5 -q "$file" | cut -c1-8)"
done
```

**Result**: One Bash call instead of N calls for N files.

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
- **Intro template**: [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md)
- **Task template**: [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)

**Guides:**
- **Enrichment checklist**: [ENRICHMENT-CHECKLIST.md](references/ENRICHMENT-CHECKLIST.md)
- **Failure modes examples**: [FAILURE-MODES-EXAMPLES.md](references/FAILURE-MODES-EXAMPLES.md)
- **Migration patterns**: [MIGRATION-PATTERNS.md](references/MIGRATION-PATTERNS.md)
- **Execution guide**: [EXECUTION-GUIDE.md](references/EXECUTION-GUIDE.md)

## Execution Handoff

After saving the enriched plan, offer:

> **Plan enriched and saved:**
> - Intro: `docs/plans/<feature>/intro.md`
> - Tasks: `docs/plans/<feature>/task-0.md` through `task-N.md`
>
> Ready to execute?
> 1. **Execute now** - Start working through tasks in this session
> 2. **New session** - Open fresh session, run `/execute-plan <feature-name>`

## INDEX.md Maintenance

After saving an enriched plan, update `docs/plans/INDEX.md`:

1. Read current INDEX.md
2. Check if plan already exists in table
3. If new: Add row to "Active Plans" with:
   - Link to plan intro file
   - Created date (from filename)
   - Task count (count task files)
   - Progress: 0/N
   - Branch (from plan's Git Context)
   - Status: ⏳ Not Started
4. If existing: Update progress and status based on Execution Log
5. Write updated INDEX.md

**INDEX.md format:**
```markdown
| Plan | Created | Tasks | Progress | Branch | Status |
|------|---------|-------|----------|--------|--------|
| [Feature A](./2026-01-14-feature-a/intro.md) | 2026-01-14 | 8 | 3/8 | feature/a | 🔄 |
| [Feature B](./2026-01-13-feature-b/intro.md) | 2026-01-13 | 2 | 0/2 | feature/b | ⏳ |
```

When a plan reaches 100% completion:
- Move from "Active Plans" to "Completed Plans"
- Add completion date
