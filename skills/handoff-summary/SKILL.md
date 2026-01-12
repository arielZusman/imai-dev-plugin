---
name: handoff-summary
description: Generate structured handoff for session transitions or task boundaries. Use before clearing session, after completing a task group, or when context degradation is suspected.
---

# Handoff Summary Generator

**Announce:** "I'm using the handoff-summary skill to create a structured handoff for the next session/task."

## When to Use

- Before clearing session (rotation recommended)
- After completing a logical task group
- When context feels degraded (repeating yourself, losing track)
- Before handing off to a different person/session

## Quick Start

1. Identify what was accomplished
2. Capture current state (files, tests, blockers)
3. Document what the next session needs to know
4. Write to checkpoint file or output directly

## Handoff Structure

Generate this structure:

```markdown
## Handoff Summary

**Generated:** [timestamp]
**Plan:** [plan file path]
**Tasks Completed:** [list of task numbers]

### What Was Done

- [Bullet list of completed work]
- [Include specific file changes]
- [Note any refactoring or cleanup]

### Current State

> Run these commands before filling in this section. Do not guess.
> ```bash
> npm run build && npm run test && git status
> ```

- **Build:** pass | fail (reason)
- **Tests:** X/Y passing (note expected failures)
- **Uncommitted Changes:** [list files or "none"]
- **Branch:** [branch name]

### Next Task

- **Task:** [number] - [title]
- **Complexity:** [simple/moderate/complex]
- **First Step:** [what to do first]

### Context Requirements

Files the next session MUST re-read:
- `path/to/file.ts` - [why: what's in it that matters]
- `path/to/other.ts` - [why]

### Decisions Made

| Decision | Rationale | Alternative Considered |
|----------|-----------|----------------------|
| [choice] | [why] | [what else was possible] |

### Gotchas

- [Non-obvious things that could trip up the next session]
- [Edge cases discovered]
- [Dependencies or order-of-operations]

### Open Questions

- [ ] [Anything unresolved that needs user input]
- [ ] [Technical uncertainties]
```

## Integration with Checkpoint

When generating a handoff, also update the checkpoint file:

1. Read existing checkpoint at `docs/plans/.state/<plan-slug>.checkpoint.md`
2. Update `execution.current_task` to next task number
3. Add handoff notes to the completed task's section
4. Append to session_log
5. Write updated checkpoint

## Output Options

**Option 1: Append to Checkpoint** (recommended for plan execution)
```
/checkpoint <plan-path> <task-number> completed
```
Then provide handoff notes when prompted.

**Option 2: Output Directly** (for ad-hoc handoffs)
Generate the handoff structure and display to user.

**Option 3: Write to File** (for complex handoffs)
Write to `docs/plans/.state/<plan-slug>.handoff-<date>.md`

## Examples

### Minimal Handoff (simple task)

```markdown
## Handoff Summary

**Tasks Completed:** Task 3

### What Was Done
- Added email validation to `src/validators.ts:45-60`
- Updated tests in `src/__tests__/validators.spec.ts`

### Current State
- Build: pass
- Tests: 24/24 passing

### Next Task
- Task 4: Add phone validation
- First Step: Copy email pattern, adapt regex
```

### Detailed Handoff (complex work)

```markdown
## Handoff Summary

**Generated:** 2026-01-08T14:30:00Z
**Plan:** docs/plans/2026-01-08-auth-feature.md
**Tasks Completed:** Tasks 5, 6, 7

### What Was Done
- Implemented JWT token generation (`src/auth/jwt.service.ts`)
- Added refresh token rotation (`src/auth/refresh.service.ts`)
- Created auth middleware (`src/middleware/auth.middleware.ts`)
- Updated 12 route handlers to use new middleware

### Current State
- Build: pass
- Tests: 45/48 passing (3 expected failures for Task 8)
- Uncommitted Changes: none (committed as abc1234)
- Branch: feature/auth-jwt

### Next Task
- Task 8: Integration tests for auth flow
- Complexity: moderate
- First Step: Set up test fixtures for user creation

### Context Requirements
- `src/auth/jwt.service.ts` - Token generation logic, expiry settings
- `src/auth/refresh.service.ts` - Rotation algorithm, revocation list
- `src/config/auth.config.ts` - All timing constants

### Decisions Made
| Decision | Rationale | Alternative Considered |
|----------|-----------|----------------------|
| HS256 over RS256 | Simpler for single-service, no key distribution | RS256 for multi-service |
| 15min access token | Balance security/UX | 1hr (too long), 5min (too short) |
| Redis for revocation | Already in stack, fast lookup | DB table (slower) |

### Gotchas
- Refresh token rotation invalidates ALL previous tokens for user
- Clock skew tolerance is 30s - don't test with time mocking > 30s
- `AUTH_SECRET` env var must be at least 32 chars

### Open Questions
- [ ] Should we add rate limiting to token refresh endpoint?
```

## Session Log Entry

When generating handoff, add to session_log:

```yaml
- timestamp: 2026-01-08T14:30:00Z
  action: handoff_generated
  tasks: [5, 6, 7]
  notes: "JWT auth complete, ready for integration tests"
```
