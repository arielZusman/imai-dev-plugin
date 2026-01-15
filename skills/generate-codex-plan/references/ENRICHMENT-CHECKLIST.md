# Enrichment Checklist (Codex Plans)

Verify each item before saving the enriched Codex plan.

## Multi-File Format Validation

Codex plans ALWAYS use multi-file format for automation support.

### Intro File (`<plan-folder>/intro.md`)
- [ ] **Format field** - `Format: multi-file (codex)` in Plan Metadata table
- [ ] **Division of Labor** - Clear statement: Codex = code, Claude Code = everything else
- [ ] **Task Index table** - Links to all task files with relative paths (`./task-N.md`)
- [ ] **No Dispatch column** - Remove sub-agent dispatch (Codex doesn't support it)
- [ ] **No Skill recommendations** - Remove skill references (Codex doesn't have access)
- [ ] **Code snippets** - All snippets in "Relevant Code Context" section (NOT in task files)
- [ ] **Execution Log** - Status table with row for each task (single source of truth)
- [ ] **Migration patterns** - In intro only (if applicable)

### Task Files (`<plan-folder>/task-N.md`)
- [ ] **Reference header** - Links back to intro file (`./intro.md`) with "See intro for" note
- [ ] **No sub-agent fields** - Remove "Dispatch" and "Recommended skill" fields
- [ ] **Codex-specific steps** - Steps clearly marked "for Codex CLI"
- [ ] **Claude Code verification** - Verification section marked "for Claude Code"
- [ ] **No duplicate code snippets** - Task files reference intro for code context
- [ ] **Self-contained execution** - All steps, verification, failure modes included
- [ ] **Checksums and test files** - Task-specific, not shared
- [ ] **Dual checklist** - Separate checklists for Codex CLI and Claude Code

### Directory Structure
- [ ] **Plan folder:** `docs/plans/codex/YYYY-MM-DD-<feature-name>/`
- [ ] **Intro file:** `intro.md` in plan folder
- [ ] **Task files:** `task-0.md`, `task-1.md`, etc. in plan folder
- [ ] **Checkpoint:** `checkpoint.md` in plan folder (created during execution)
- [ ] **Task 0** - Prerequisites verification task exists (optional but recommended)

---

## Required Content

### Plan Structure
- [ ] **Project context** - Repo, service, branch
- [ ] **Fresh Session Entry Point** - Reading order for session resume
- [ ] **Goal** - One sentence summary
- [ ] **Architecture** - Design approach in prose (2-3 sentences, avoid bullets)
- [ ] **Execution Workflow section** - At TOP with Codex + Claude Code split clearly defined
- [ ] **Relevant code context** - Key snippets that Codex needs
- [ ] **Migration patterns** - For upgrade plans, include before/after examples

### Per-Task Requirements
- [ ] **Task 0 for prerequisites** - Explicit verification before Task 1
- [ ] **Why field with Business + Technical** - Both contexts explained
- [ ] **Time estimate** - ~15 min (🟢), ~30 min (🟡), ~1 hr (🔴)
- [ ] **Exact file paths** - With line numbers for modifications
- [ ] **Context Requirements** - REQUIRED vs REFERENCE files
- [ ] **Test file discovery** - Related .spec.ts files with mock locations
- [ ] **File checksums** - For modified files, enables skip-if-already-done
- [ ] **Clear intent or exact content** - Intent for logic, exact content for templates/config
- [ ] **Step granularity** - Each step is ONE action for Codex
- [ ] **Commands with expected output** - For Claude Code verification
- [ ] **Verification per task** - How Claude Code confirms completion
- [ ] **Failure modes with Rollback** - Include note to use Codex for code fixes
- [ ] **Handoff notes placeholder** - Space to document decisions
- [ ] **Dual checklist** - Codex tasks + Claude Code tasks separated

### Task Coordination
- [ ] **Task dependencies** - "Depends on" field for each task
- [ ] **Parallel groups** - Independent tasks grouped for parallel Codex instances
- [ ] **Complexity indicators** - Each task marked 🟢/🟡/🔴
  - 🟢 Simple: < 50 lines, single file
  - 🟡 Moderate: 50-200 lines, multiple files
  - 🔴 Complex: > 200 lines, architectural
- [ ] **NO sub-agent recommendations** - Removed (Codex doesn't support)
- [ ] **NO skill recommendations** - Removed (Codex doesn't have access)

### Plan-Level
- [ ] **Task execution order** - Sequential and parallel groups defined
- [ ] **Final checklist** - Tests, build, feature verification
- [ ] **Gotchas** - Non-obvious issues
- [ ] **Review protocol** - Per-task review scope (handled by Claude Code)
  - Aspect selection: `code` (always) + `errors` (if error handling) + `types` (if types modified) + `tests` (if tests added)
- [ ] **Execution Log** - Table with row per task, all marked ⏳ Pending

## Content Guidelines

**Include exact content when:**
- Translation keys (exact strings matter)
- HTML templates (structure must be exact)
- Config values (specific values required)
- Simple boilerplate (pattern is exact copy)

**Use intent + location when:**
- Business logic (Codex can derive from context)
- Complex functions (needs adaptation to current code)
- Conditional logic (context-dependent)

**Rule of thumb:** Include exact content if Codex needs the *exact characters*. Use intent if Codex can derive it from context.

---

## Why Field Guidelines

Each task should explain why it matters, not just what to do.

**Format:**
```markdown
**Why this task matters:**
- **Business:** [How this serves the user/product goal]
- **Technical:** [Why this approach/order is correct]
```

**Good examples:**
- Business: "Enables users to filter search results by date range"
- Technical: "Must complete before Task 3 (search UI) to provide required API"

**Bad examples:**
- ❌ "Needed for the feature" (too vague)
- ❌ "Part of the plan" (not explaining why)

---

## Key Differences from Regular Plans

1. **No sub-agent dispatch** - All code changes go to Codex
2. **No skill recommendations** - Codex doesn't have access to Claude Code skills
3. **Always multi-file** - Even for small plans (supports automation)
4. **Clear responsibility split** - Codex vs Claude Code clearly marked
5. **Dual checklists** - Separate tasks for Codex and Claude Code
6. **Output location** - `docs/plans/codex/` instead of `docs/plans/`
