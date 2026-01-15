# Codex Execution Plans Index

This directory contains enriched implementation plans optimized for Codex CLI execution.

## About Codex Plans

**Division of Labor:**
- **Codex CLI:** Handles all code changes (reading files, editing code, creating files)
- **Claude Code:** Handles checkpoints, verification, code review, commits, tests

**Format:**
- All plans use multi-file format to support automation
- Each plan has its own directory: `YYYY-MM-DD-<feature-name>/`
- Directory contains: `intro.md` + `task-N.md` files

**Execution:**
- Each task file can be passed to Codex CLI independently
- Enables scripting automation to iterate through tasks
- See intro.md in each plan for execution workflow

---

## Active Plans

| Plan | Format | Created | Tasks | Progress | Branch | Status |
|------|--------|---------|-------|----------|--------|--------|
| _(none yet)_ | - | - | - | - | - | - |

**Status Legend:**
- ⏳ Not Started
- 🔄 In Progress
- ✅ Completed
- ⛔ Blocked

---

## Completed Plans

| Plan | Format | Created | Tasks | Completed | Branch |
|------|--------|---------|-------|-----------|--------|
| _(none yet)_ | - | - | - | - | - |

---

## How to Create a Codex Plan

1. Create a basic plan outline
2. Run `/generate-codex-plan <plan-path>` to enrich it
3. The enriched plan will be saved to this directory
4. Each task can be executed independently with Codex CLI

## How to Execute a Codex Plan

### Manual Execution:
1. Read `<plan-name>/intro.md`
2. For each task:
   - Load task file (e.g., `task-1.md`)
   - Use Codex CLI to implement code changes
   - Use Claude Code for verification and review
   - Commit and checkpoint

### Automated Execution:
Create a script that iterates through task files and coordinates Codex + Claude Code. See EXECUTION-GUIDE.md in the skill for examples.
