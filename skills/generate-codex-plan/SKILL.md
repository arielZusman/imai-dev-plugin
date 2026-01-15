---
name: generate-codex-plan
description: |
  Converts basic plans into Codex CLI execution-ready documents with embedded context,
  verification steps, and mandatory code review workflow.

  Use after creating a plan, or when preparing plans for Codex CLI execution
  where inline context is essential and execution is split into multiple files
  for automation.
metadata:
  context: fork
  agent: Plan
---

# Generate Codex Plan

**Announce:** "I'm using the generate-codex-plan skill to enrich this plan for Codex CLI execution. This will create a multi-file structure optimized for automated execution where Codex handles code changes and Claude Code handles everything else."

## Quick Start

1. Identify plan from `~/.claude/plans/` or `~/.codex/plans/`
2. Read and gather context for referenced files
3. **Output format:**
   - **Multi-file (default):** Always use [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md) + [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)
   - Each task in a separate file for automation support
   - Task 0 (prerequisites) is optional but recommended
4. Save to `docs/plans/codex/`:
   - Create folder `YYYY-MM-DD-<feature-name>/` containing `intro.md` + `task-N.md` files
5. Update `docs/plans/codex/INDEX.md`

## Process

1. **Identify the plan** - Ask user which plan or use the most recent
2. **Read the plan** - Understand tasks and scope
3. **Gather context** - Read relevant files mentioned in the plan
4. **Discover test files** - For each modified file, find related `.spec.ts` files and mock locations
5. **Compute checksums** - Calculate md5 checksums for files to be modified (first 8 chars)
6. **Gather dependency info** - For package upgrades, query peer dependencies and known issues
7. **Enrich** - Add self-contained header and inline code context
8. **Add verification** - How to confirm each task is complete
9. **Generate ALL output content at once:**

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
      - Standard verification steps
      - Common failure modes

   c) **Generation strategy:**
      - All Sonnet sections first (complex analysis)
      - All template substitutions second (simple replacement)
      - All Haiku sections last (parallel sub-agents for boilerplate)

   **Multi-file format (always):**
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

10. **Write ALL files in parallel:**

    **CRITICAL OPTIMIZATION**: Use a SINGLE message with MULTIPLE Write tool calls.
    Do NOT write files sequentially.

    **Multi-file format:**
    First, create the directory if needed:
    ```markdown
    Creating directory and writing all plan files in parallel:
    ```

    Then make ONE tool call message containing ALL Write operations:
    - Write(docs/plans/codex/YYYY-MM-DD-<feature>/intro.md, INTRO_CONTENT)
    - Write(docs/plans/codex/YYYY-MM-DD-<feature>/task-0.md, TASK_0_CONTENT)
    - Write(docs/plans/codex/YYYY-MM-DD-<feature>/task-1.md, TASK_1_CONTENT)
    - Write(docs/plans/codex/YYYY-MM-DD-<feature>/task-2.md, TASK_2_CONTENT)
    - ... (all task files)

    **Why**: Parallel writes execute simultaneously. Sequential writes wait for each API roundtrip (~26-53s per file).

11. **Update INDEX** - Add/update entry in `docs/plans/codex/INDEX.md`

## Key Principles

The enriched plan is for **Codex CLI execution** with the following division of labor:

- **Codex handles:** Code changes only (reading files, editing code, creating new files)
- **Claude Code handles:** Everything else (checkpoints, reviews, verification, commits, tests)

The plan is split into multiple files to support future automation where a script can:
1. Iterate through task files
2. Pass each task to Codex CLI for code implementation
3. Use Claude Code for verification and review
4. Move to next task automatically

## Multi-File Format: Content Split

Content is split to enable automated execution per task:

| Content | Location | Rationale |
|---------|----------|-----------|
| Project Context | Intro only | Shared metadata |
| Architecture | Intro only | Design decisions are global |
| Code Snippets | Intro only | Avoids duplicating 50-100 lines per task |
| Migration Patterns | Intro only | Reference patterns apply to all tasks |
| Execution Workflow | Intro only | Process is the same for all tasks |
| Execution Log | Intro only | Single source of truth for status |
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

- Include all context inline (Codex has no memory of previous conversation)
- Be explicit about file locations and patterns
- Reference relevant code snippets directly in the plan
- Intent for business logic, exact content for templates/config
- Verification criteria for every task

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

**Note for Codex plans:** Complexity is informational only. All tasks are passed to Codex CLI for implementation.

**Who assigns:** Plan enricher during enrichment process (not the original planner)

**Validation:** Plan validator checks all tasks have complexity assigned

## Critical: Execution Workflow Embedding

The enriched plan MUST include these elements to ensure code reviews are not skipped:

1. **Execution Workflow section** - Placed BEFORE tasks, not buried at the end
2. **Per-task checklist** - Each task ends with a checklist including the review step

This workflow mandates:
- **Codex handles:** Code implementation only
- **Claude Code handles:** Checkpoints, verification, code review (`/pr-review-toolkit:review-pr`), commits
- Status updates in Execution Log
- Fix-and-retry cycle for critical issues (max 2 cycles)
- Session handoff protocol

Without these, executors will skip reviews because the instructions are too far from where they're working.

## Key Differences from generate-implementation-plan

1. **No sub-agent references** - Codex CLI doesn't support Claude Code sub-agents
2. **No skill recommendations** - Codex doesn't have access to Claude Code skills
3. **Simplified dispatch** - Only "codex" dispatch type (Codex handles all code changes)
4. **Always multi-file** - To support future automation scripts
5. **Clear responsibility split** - Codex = code, Claude Code = everything else
6. **Output location** - `docs/plans/codex/` instead of `docs/plans/`

## References

**Templates:**
- **Multi-file intro template**: [INTRO-TEMPLATE.md](assets/INTRO-TEMPLATE.md)
- **Multi-file task template**: [TASK-TEMPLATE.md](assets/TASK-TEMPLATE.md)

**Guides:**
- **Enrichment checklist**: [ENRICHMENT-CHECKLIST.md](references/ENRICHMENT-CHECKLIST.md)
- **Failure modes examples**: [FAILURE-MODES-EXAMPLES.md](references/FAILURE-MODES-EXAMPLES.md)
- **Migration patterns**: [MIGRATION-PATTERNS.md](references/MIGRATION-PATTERNS.md)
- **Execution guide**: [EXECUTION-GUIDE.md](references/EXECUTION-GUIDE.md)

## Execution Handoff

After saving the enriched plan, offer:

> **Codex plan enriched and saved:**
> - Intro: `docs/plans/codex/<feature>/intro.md`
> - Tasks: `docs/plans/codex/<feature>/task-0.md` through `task-N.md`
>
> Ready to execute?
> 1. **Execute with Codex** - Each task file can be passed to Codex CLI for code implementation
> 2. **Manual execution** - Work through tasks manually, using Codex for code changes
> 3. **Future automation** - Script can iterate through task files automatically

## INDEX.md Maintenance

After saving an enriched plan, update `docs/plans/codex/INDEX.md`:

1. Read current INDEX.md (create if doesn't exist)
2. Check if plan already exists in table
3. If new: Add row to "Active Plans" with:
   - Link to intro file
   - **Format** column: `multi`
   - Created date (from filename)
   - Task count (count task files)
   - Progress: 0/N
   - Branch (from plan's Git Context)
   - Status: ⏳ Not Started
4. If existing: Update progress and status based on Execution Log
5. Write updated INDEX.md

**INDEX.md format:**
```markdown
| Plan | Format | Created | Tasks | Progress | Branch | Status |
|------|--------|---------|-------|----------|--------|--------|
| [Feature A](./2026-01-14-feature-a/intro.md) | multi | 2026-01-14 | 8 | 3/8 | feature/a | 🔄 |
| [Feature B](./2026-01-13-feature-b/intro.md) | multi | 2026-01-13 | 2 | 0/2 | feature/b | ⏳ |
```

When a plan reaches 100% completion:
- Move from "Active Plans" to "Completed Plans"
- Add completion date
