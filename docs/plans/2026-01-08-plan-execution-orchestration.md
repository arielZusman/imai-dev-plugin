# Plan Execution Orchestration Improvements Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Reduce manual session cycling and context rot while preserving task context across plan execution.

**Architecture:** Orchestrator-driven workflow with repo-local state files in `docs/agent-state/` and lightweight Claude Code commands for priming, resuming, summarizing, and handoff. Commands generate structured task packets and enforce small context budgets.

**Tech Stack:** Claude Code commands (Markdown), repo-local Markdown state files, git.

---

### Task 1: Add agent-state README

**Files:**
- Create: `docs/agent-state/README.md`

**Step 1: Write the README**

```markdown
# Agent State Files

This directory stores persistent context to reduce session re-priming and context rot.

## Files
- `plan-summary.md`: Current plan index, current task, next actions.
- `task-template.md`: Template for per-task briefs.
- `task-<id>.md`: Per-task brief and handoff report.
- `session-log.md`: Rotating log of the last 20 session summaries.

## Conventions
- Keep per-task summaries under 12 lines.
- Use ISO timestamps (UTC) for log entries.
- Include a context budget for each task (tokens).
- Update `plan-summary.md` after every task completion.

## Rotation
- `session-log.md` should keep only the 20 most recent entries. Delete older entries when adding new ones.
```

**Step 2: Verify file exists**

Run: `rg -n "Agent State Files" docs/agent-state/README.md`
Expected: A match on line 1.

**Step 3: Commit**

```bash
git add docs/agent-state/README.md
git commit -m "docs: add agent state README"
```

---

### Task 2: Add plan summary template

**Files:**
- Create: `docs/agent-state/plan-summary.md`

**Step 1: Write the template**

```markdown
# Plan Summary

Project: <repo name>
Plan file: <path>
Last updated: <YYYY-MM-DD>

## Task Index
| ID | Status | Title | Depends On | Owner | Context Budget |
| --- | --- | --- | --- | --- | --- |
| T1 | pending | Example task title | - | orchestrator | 1200 |

## Current Task
- ID: <T1>
- Goal: <one sentence>
- Required files: <list>
- Verification: <command and expected output>
- Risks: <short list>

## Next Actions
- <short, ordered list>

## Notes
- <anything the next session must know>
```

**Step 2: Verify file exists**

Run: `rg -n "Task Index" docs/agent-state/plan-summary.md`
Expected: A match on the header line.

**Step 3: Commit**

```bash
git add docs/agent-state/plan-summary.md
git commit -m "docs: add plan summary template"
```

---

### Task 3: Add per-task template

**Files:**
- Create: `docs/agent-state/task-template.md`

**Step 1: Write the template**

```markdown
# Task <ID>: <Title>

Status: <pending|in_progress|done|blocked>
Owner: <orchestrator|implementer|reviewer|tester>
Depends On: <task ids>
Context Budget: <tokens>

## Goal
<one sentence>

## Required Files
- <path>

## Preconditions
- <precondition>

## Steps
1. <one action>
2. <one action>

## Verification
- Run: <command>
- Expected: <result>

## Risks
- <risk>

## Handoff
- Changes: <summary>
- Files touched: <list>
- Commands run: <list>
- Results: <pass/fail>
- Follow-ups: <list>
```

**Step 2: Verify file exists**

Run: `rg -n "Handoff" docs/agent-state/task-template.md`
Expected: A match on the section header.

**Step 3: Commit**

```bash
git add docs/agent-state/task-template.md
git commit -m "docs: add task template"
```

---

### Task 4: Add session log template

**Files:**
- Create: `docs/agent-state/session-log.md`

**Step 1: Write the template**

```markdown
# Session Log

- 2026-01-08T00:00:00Z | Session started | Notes: <short summary>
```

**Step 2: Verify file exists**

Run: `rg -n "Session Log" docs/agent-state/session-log.md`
Expected: A match on line 1.

**Step 3: Commit**

```bash
git add docs/agent-state/session-log.md
git commit -m "docs: add session log template"
```

---

### Task 5: Add generate-implementation-plan command

**Files:**
- Create: `autocoder/.claude/commands/generate-implementation-plan.md`

**Step 1: Write the command file**

```markdown
---
description: Generate an enriched implementation plan with task packets
---

# INPUTS
This command requires a source plan or spec path via `$ARGUMENTS`.
If `$ARGUMENTS` is empty, ask for the path and stop.

# OUTPUT
Write the enriched plan to `docs/plans/YYYY-MM-DD-<topic>.md` unless a specific output path is included in `$ARGUMENTS`.

# PROCESS
1. Read the source file.
2. Produce an enriched plan with the required header format.
3. Decompose into tasks with explicit dependencies and context budgets.
4. Include verification commands and expected results for each task.
5. Add a short success metrics section at the end.

# REQUIRED HEADER
```
# <Feature Name> Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** <one sentence>

**Architecture:** <2-3 sentences>

**Tech Stack:** <key technologies>

---
```

# TASK PACKET TEMPLATE
```
### Task N: <Title>

**Files:**
- Create: `path`
- Modify: `path:line`
- Test: `path`

**Context Budget:** <tokens>
**Depends On:** <task ids>
**Definition of Done:** <short checklist>

**Step 1: Write the failing test**
<test code>

**Step 2: Run test to verify it fails**
Run: <command>
Expected: FAIL with <message>

**Step 3: Write minimal implementation**
<code>

**Step 4: Run test to verify it passes**
Run: <command>
Expected: PASS

**Step 5: Commit**
<git add + git commit>
```

# FOOTER
Include:
- Task execution order
- Quick wins vs. larger refactors
- Success metrics
```

**Step 2: Verify file exists**

Run: `rg -n "TASK PACKET TEMPLATE" autocoder/.claude/commands/generate-implementation-plan.md`
Expected: A match on the header line.

**Step 3: Commit**

```bash
git add autocoder/.claude/commands/generate-implementation-plan.md
git commit -m "feat: add generate-implementation-plan command"
```

---

### Task 6: Add prime-plan command

**Files:**
- Create: `autocoder/.claude/commands/prime-plan.md`

**Step 1: Write the command file**

```markdown
---
description: Prime a session with the current plan and task context
---

# INPUTS
Optional `$ARGUMENTS`: task id (e.g., `T3`).

# STEPS
1. Show repo context:
   - `!git branch --show-current`
   - `!git status --short`
   - `!git log --oneline -5`
2. Read `docs/agent-state/plan-summary.md`.
3. If a task id is provided, read `docs/agent-state/task-<id>.md`.
   Otherwise, read the "Current Task" section from `plan-summary.md`.
4. Output a compact briefing with:
   - Current task goal
   - Required files
   - Verification command
   - Next action

# OUTPUT FORMAT
```
Session Context
- Branch: <name>
- Status: <clean|dirty>
- Recent commits: <list>

Task Brief
- ID: <id>
- Goal: <one sentence>
- Required files: <list>
- Verification: <command>
- Next action: <one step>
```
```

**Step 2: Verify file exists**

Run: `rg -n "Task Brief" autocoder/.claude/commands/prime-plan.md`
Expected: A match on the output format header.

**Step 3: Commit**

```bash
git add autocoder/.claude/commands/prime-plan.md
git commit -m "feat: add prime-plan command"
```

---

### Task 7: Add resume-task command

**Files:**
- Create: `autocoder/.claude/commands/resume-task.md`

**Step 1: Write the command file**

```markdown
---
description: Resume a specific task with minimal context
---

# INPUTS
This command requires a task id via `$ARGUMENTS`.
If `$ARGUMENTS` is empty, ask for the task id and stop.

# STEPS
1. Read `docs/agent-state/task-<id>.md`.
2. Read each file listed in "Required Files".
3. Summarize open risks and verification steps.
4. Ask for confirmation before implementing.

# OUTPUT FORMAT
```
Task Ready
- ID: <id>
- Goal: <one sentence>
- Required files loaded: <list>
- Risks: <list>
- Verification: <command>

Ready to implement? (yes/no)
```
```

**Step 2: Verify file exists**

Run: `rg -n "Task Ready" autocoder/.claude/commands/resume-task.md`
Expected: A match on the output format header.

**Step 3: Commit**

```bash
git add autocoder/.claude/commands/resume-task.md
git commit -m "feat: add resume-task command"
```

---

### Task 8: Add summarize-session command

**Files:**
- Create: `autocoder/.claude/commands/summarize-session.md`

**Step 1: Write the command file**

```markdown
---
description: Summarize the session into agent-state files
---

# STEPS
1. Update `docs/agent-state/plan-summary.md` with task status changes.
2. Update `docs/agent-state/task-<id>.md` for any task touched in this session.
3. Append a new entry to `docs/agent-state/session-log.md`.
4. Keep only the 20 most recent log entries.

# OUTPUT TEMPLATE
```
Summary Written
- Updated: plan-summary.md
- Updated: task-<id>.md
- Appended: session-log.md
- Log entries kept: 20
```
```

**Step 2: Verify file exists**

Run: `rg -n "Summary Written" autocoder/.claude/commands/summarize-session.md`
Expected: A match on the output template header.

**Step 3: Commit**

```bash
git add autocoder/.claude/commands/summarize-session.md
git commit -m "feat: add summarize-session command"
```

---

### Task 9: Add handoff command

**Files:**
- Create: `autocoder/.claude/commands/handoff.md`

**Step 1: Write the command file**

```markdown
---
description: Write a structured handoff report for a task
---

# INPUTS
This command requires a task id via `$ARGUMENTS`.
If `$ARGUMENTS` is empty, ask for the task id and stop.

# STEPS
1. Open `docs/agent-state/task-<id>.md`.
2. Fill in the "Handoff" section with:
   - Changes
   - Files touched
   - Commands run
   - Results
   - Follow-ups
3. Update the task status in `plan-summary.md`.

# OUTPUT TEMPLATE
```
Handoff Complete
- Task: <id>
- Updated: task-<id>.md
- Updated: plan-summary.md
```
```

**Step 2: Verify file exists**

Run: `rg -n "Handoff Complete" autocoder/.claude/commands/handoff.md`
Expected: A match on the output template header.

**Step 3: Commit**

```bash
git add autocoder/.claude/commands/handoff.md
git commit -m "feat: add handoff command"
```

---

## Execution Notes

- Keep task context under the specified budget in each task packet.
- Use @superpowers:executing-plans when implementing.
- Use @superpowers:subagent-driven-development if executing in this session.

## Success Metrics

- Manual steps per task cycle reduced to 2 or fewer.
- Task context size reduced by 60-80 percent compared to full plan load.
- Fewer restarts or re-priming steps per task.

