# Checkpoint File Format Specification

Checkpoint files track plan execution state across sessions. They enable:
- Fast session resumption without re-reading full plan
- Task-level state persistence
- Session debugging via rolling log
- Handoff context between tasks

## File Location

```
docs/plans/.state/<plan-slug>.checkpoint.md
```

Where `<plan-slug>` is derived from the plan filename (e.g., `2026-01-08-hashtag-request-budget-strategy`).

## Format

Markdown with YAML frontmatter for ecosystem consistency.

```markdown
---
plan_file: docs/plans/2026-01-08-hashtag-request-budget-strategy.md
created_at: 2026-01-08T01:52:00Z
updated_at: 2026-01-08T06:45:00Z

execution:
  current_task: 9
  current_phase: implement  # load | implement | verify | review | checkpoint
  status: in_progress       # in_progress | completed | blocked
  tasks_completed_this_session: 2

rotation_heuristic:
  task_count: 2
  session_start: 2026-01-08T06:00:00Z
  recommend_rotation: false  # true when task_count >= 4 or elapsed > 30min
---

# Checkpoint: [Plan Name]

## Current Position

- **Task:** 9 - Two-Phase Implementation
- **Phase:** implement
- **Status:** in_progress

## Completed Tasks

### Task 1: Define Interfaces
- **Status:** done
- **Outputs:**
  - `src/interfaces/budget-config.interface.ts`
- **Verification:** build pass
- **Committed:** abc1234

### Task 8: Write Two-Phase Tests
- **Status:** done
- **Outputs:**
  - `src/__tests__/tiktok-hashtag-fetch.service.spec.ts`
  - Tests written: 7
- **Verification:** tests 5 fail (expected), build pass
- **Committed:** def5678
- **Handoff Notes:**
  Tests expect `processHashtagPhase1()` and `processHashtagPhase2()` methods.
  Phase 1 must return `{ posts, yield, nextCursor }`.
  Phase 2 filters by yield threshold (default 0.7).

## Context Requirements for Current Task

### Required (must re-read)
- `src/services/tiktok-hashtag-fetch.service.ts` - sections: processHashtag, fetchForHashtags
- `src/interfaces/budget-config.interface.ts` - all

### Reference (consult if needed)
- `docs/plans/2026-01-08-hashtag-request-budget-strategy.md` - sections: Architecture, Phase 1 Algorithm, Phase 2 Algorithm

## Modified Files (This Plan)

- `src/services/tiktok-hashtag-fetch.service.ts`
- `src/__tests__/tiktok-hashtag-fetch.service.spec.ts`
- `src/interfaces/budget-config.interface.ts`

## Last Verification

- **Build:** pass
- **Tests:** 27/32 passing (5 expected failures)
- **Timestamp:** 2026-01-08T01:57:44Z

## Session Log

| Timestamp | Action | Task | Notes |
|-----------|--------|------|-------|
| 2026-01-08T06:00:00Z | session_started | - | Resumed from checkpoint |
| 2026-01-08T06:05:00Z | task_started | 9 | Verified 7 tests exist |
| 2026-01-08T06:15:30Z | error | 9 | Build failed - missing import for BudgetConfig |
| 2026-01-08T06:17:44Z | error_resolved | 9 | Added missing import |
| 2026-01-08T06:25:00Z | task_completed | 9 | build pass, tests 5 fail (expected) |
```

## Field Descriptions

### Frontmatter

| Field | Type | Description |
|-------|------|-------------|
| `plan_file` | string | Path to the enriched plan file |
| `created_at` | ISO 8601 | When checkpoint was first created |
| `updated_at` | ISO 8601 | Last modification time |
| `execution.current_task` | number | Task number currently in progress |
| `execution.current_phase` | enum | `load`, `implement`, `verify`, `review`, `checkpoint` |
| `execution.status` | enum | `in_progress`, `completed`, `blocked` |
| `execution.tasks_completed_this_session` | number | Counter for rotation heuristic |
| `rotation_heuristic.task_count` | number | Tasks completed this session |
| `rotation_heuristic.session_start` | ISO 8601 | When current session began |
| `rotation_heuristic.recommend_rotation` | boolean | True when rotation recommended |

### Body Sections

| Section | Purpose |
|---------|---------|
| Current Position | Quick reference for where to resume |
| Completed Tasks | Per-task results with handoff notes |
| Context Requirements | Files to re-read for current task |
| Modified Files | All files changed by this plan |
| Last Verification | Most recent build/test status |
| Session Log | Rolling log (keep last 10-20 entries) |

## Session Log Actions

| Action | When to Log |
|--------|-------------|
| `session_started` | New session begins, checkpoint loaded |
| `task_started` | Beginning work on a task |
| `task_completed` | Task finished successfully |
| `error` | Any error encountered (build fail, test fail, etc.) |
| `error_resolved` | Error was fixed |
| `blocked` | Task cannot proceed |
| `rotation_recommended` | Heuristic triggered rotation suggestion |

## Rotation Heuristic

Recommend session rotation when ANY of:
- `tasks_completed_this_session >= 4`
- `now - session_start > 30 minutes`
- User reports confusion or degraded responses

When rotation is recommended:
1. `/checkpoint` outputs warning
2. User can override and continue
3. If rotating: clear session, run `/resume-plan <plan-path>`

## Usage

### Creating a Checkpoint

```
/checkpoint <task-number> <status> [notes]
```

### Resuming from Checkpoint

```
/resume-plan <plan-path> [task-id]
```

If checkpoint exists, loads task-specific context instead of full plan.

### Manual Inspection

Checkpoint files are human-readable Markdown. Open in any editor to inspect state.
