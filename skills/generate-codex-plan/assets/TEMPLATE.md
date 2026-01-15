# Single-File Codex Plan Template (Legacy)

**Note:** This is a legacy format. The multi-file format is preferred for Codex plans to support automation.

Use this only if specifically requested by the user or for very simple plans (1-2 tasks).

---

```markdown
# [Feature Name] Implementation Plan (Codex)

<execution_context>
Execute this plan task-by-task using Codex CLI for code changes.

**Division of Labor:**
- **Codex CLI:** All code changes (reading files, editing code, creating files)
- **Claude Code:** Checkpoints, verification, code review, commits, tests
</execution_context>

## Plan Metadata

| Field | Value |
|-------|-------|
| Format | single-file (codex) |
| Created | YYYY-MM-DD |
| Task Count | N |
| Branch | [feature branch name] |
| Original Plan | `~/.claude/plans/[plan-filename].md` or `~/.codex/plans/[plan-filename].md` |

## Project Context

**Repository:** [repo name and path]
**Service:** [which service/module this affects]
**Target Directory:** [absolute path to working directory]

## Goal

[One sentence describing what this builds]

## Architecture

[2-3 sentences describing approach and design decisions]

---

## Execution Workflow

<workflow>
Follow this workflow for each task:

**For Codex CLI:**
1. **Read task section** below
2. **Context Load:** Read files listed in task's "Context Requirements"
3. **Implement:** Use Codex CLI to complete the code changes
   - Codex handles: reading files, editing code, creating new files

**For Claude Code (after Codex completes):**
4. **Checkpoint Start:** `/checkpoint <plan-path> <task-number> started`
5. **Verify:** Run the task's verification commands
6. **Review:** Run `/pr-review-toolkit:review-pr` with appropriate aspects
7. **Fix if needed:** Max 2 review cycles, use Codex for code fixes
8. **Commit:** After review passes
9. **Checkpoint Complete:** `/checkpoint <plan-path> <task-number> completed`
</workflow>

---

## Task 0: Verify Prerequisites

**Complexity:** 🟢 Simple

### Steps
1. [Prerequisite check commands]

### Verification
```bash
npm run build && npm run test
```

---

## Task 1: [Title]

**Complexity:** 🟡 Moderate | 🔴 Complex | 🟢 Simple
**Depends on:** Task 0

### Context Requirements
- `path/to/file.ts` - [sections to read]

### Files
- Modify: `path/to/file.ts:45-60`
- Checksum: `a1b2c3d4`

### Steps (for Codex CLI)
1. [Step 1]
2. [Step 2]

### Verification (for Claude Code)
```bash
npm run test -- [pattern]
```

### Checklist
- [ ] `/checkpoint started`
- [ ] Codex implementation complete
- [ ] Verification passed
- [ ] Review: `/pr-review-toolkit:review-pr staged code`
- [ ] Committed
- [ ] `/checkpoint completed`

---

[Repeat for each task]

---

## Execution Log

| Task | Status | Review | Committed | Notes |
|------|--------|--------|-----------|-------|
| Task 0 | ⏳ | | | |
| Task 1 | ⏳ | | | |

**Status Legend:** ⏳ Pending | 🔄 In Progress | ✅ Done | ⛔ Blocked
```
