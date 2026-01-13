---
name: session-management
description: Guidelines for managing session health, detecting context rot, and rotating sessions. Use when sessions feel degraded, after completing multiple tasks, or when planning long implementation work.
---

# Session Management & Rotation Heuristics

**Announce:** "I'm using the session-management skill to evaluate session health and rotation needs."

## Why Sessions Degrade

Claude Code sessions degrade over time due to:

1. **Context window filling** - More history = less attention to recent content
2. **Lost in the middle** - Information in middle positions gets less attention (U-shaped curve)
3. **Accumulated noise** - Error messages, debug output, exploration artifacts
4. **State drift** - Mental model diverges from actual codebase state

**Research finding:** Replacing 113k-token history with focused 300-token context boosted accuracy by 30%.

## Claude 4.5 Context Awareness

Claude 4.5 can track its remaining token budget. When approaching limits:
- Save progress to checkpoint before context compacts
- Do not artificially stop work early due to budget concerns
- Context will be automatically compacted, allowing continuation from where you left off

This means you can work persistently on long tasks. The rotation heuristics below help maintain quality, not just manage token limits.

## Session Start: Plan/Checkpoint Detection

When starting a session, check for existing plans and checkpoints:

```
On session start:
1. Check docs/plans/ for enriched plan files
2. Check docs/plans/.state/ for checkpoint files
3. If checkpoint exists with incomplete tasks:
   → Output: "Resume available: /execute-plan <plan-name>"
4. If plan exists but no checkpoint:
   → Output: "Plan ready: /execute-plan <plan-path>"
5. If multiple plans/checkpoints:
   → List all with status, let user choose
```

This ensures continuity across sessions.

## Rotation Strategy: One Task Per Session

**Stop after EACH task.** This is not optional.

| Event | Action |
|-------|--------|
| **Task completed** | Save checkpoint → Code review → Commit → **STOP** |
| **Major error recovery** | 3+ failed attempts → Rotate immediately |
| **Context feels wrong** | Repeating yourself, wrong assumptions → Rotate immediately |

### Why One Task Per Session?

- **Fresh context** - Each task starts with focused attention
- **Code review cannot be skipped** - Stopping forces the review step
- **Checkpoint always current** - No state loss on session clear
- **Failures isolated** - One task's errors don't compound

This aligns with Ralph's core insight: tight scope + backpressure + fresh context = reliable execution.

## Symptoms of Context Rot

Watch for these signs that session quality is degrading:

### Code Quality Symptoms
- Forgetting imports you added earlier
- Re-implementing functions that already exist
- Using wrong variable names from earlier context
- Missing type errors that should be obvious
- Inconsistent naming conventions within session

### Behavioral Symptoms
- Asking about code you already read
- Proposing approaches already rejected
- Losing track of which tasks are done
- Needing multiple attempts for simple changes
- Explanations becoming longer/more verbose

### User-Reported Symptoms
- "You already did that"
- "That's not what I asked"
- "We discussed this earlier"
- "Why are you repeating yourself?"

## Rotation Protocol

<mandatory_rotation_protocol>
Do not skip these steps when rotating. Incomplete handoffs cause context loss.

### Before Rotating

1. **Save checkpoint:**
   ```
   /checkpoint <plan-path> <task-number> completed
   ```
   Include detailed handoff notes.

2. **Verify state:**
   ```bash
   npm run build  # Should pass
   npm run test   # Note expected failures
   git status     # Should be clean (committed)
   ```

3. **Document position:**
   - Current task number
   - Phase within task (implement/verify/review)
   - Any open questions or blockers

### Rotating

1. **Clear session** (in Claude Code: `/clear` or start new session)

2. **In fresh session, run:**
   ```
   /execute-plan <plan-name>
   ```
   This loads checkpoint and provides focused context.

### After Rotating

1. **Verify context loaded:**
   - Checkpoint state shown
   - Required files re-read
   - Pre-conditions verified

2. **Confirm understanding:**
   - Review handoff notes
   - Check recent session_log entries
   - Verify you know what to do next
</mandatory_rotation_protocol>

## Heuristic Decision Tree

```
Session started
    │
    ├─ Task completed? ──→ STOP (mandatory)
    │   │
    │   └─ Checkpoint saved?
    │       ├─ Yes ──→ Output resume instructions
    │       └─ No ──→ ERROR: Save checkpoint first
    │
    ├─ Error occurred?
    │   │
    │   ├─ 3+ consecutive failures? ──→ ROTATE immediately
    │   └─ Recoverable error? ──→ Fix, then complete task
    │
    └─ User reports confusion? ──→ ROTATE immediately
```

**Note:** The old "4 tasks / 30 min" heuristic is replaced by "one task per session". This is simpler and more reliable.

## Task Budgeting

**One task per session.** This simplifies planning:

| Plan Size | Sessions Needed |
|-----------|-----------------|
| 5 tasks | 5 sessions |
| 10 tasks | 10 sessions |
| 20 tasks | 20 sessions |

**Planning example:**
```
Plan: 12 tasks

Sessions needed: 12
Each session:
1. /execute-plan <plan>
2. Execute one task
3. Code review
4. Commit
5. Checkpoint + STOP

Total time: ~10-15 min per session × 12 = 2-3 hours
```

This seems like more overhead, but it:
- Eliminates context degradation issues
- Ensures every task gets proper review
- Creates natural breakpoints for the user

## Integration with Commands

### /checkpoint

Reports task completion and session stop:
```
✓ Checkpoint persisted: docs/plans/.state/<plan>.checkpoint.md
- Task: 3
- Status: completed

✓ Task 3 complete. Session will pause for context refresh.

To continue: /execute-plan <plan-name>
Next task: 4 - [title]
```

### /execute-plan (resuming)

Starts fresh session with context:
```
Session started fresh.
Plan: <plan-name>
Current task: 4 of 12

Task 4: [title]
Context loaded. Ready to execute.
```

### /execute-plan

Stops after each task:
```
✓ Task 3 complete.

Session will pause for context refresh.

To continue with next task:
  /execute-plan <plan-name>

Progress: 3/12 tasks complete
Next task: 4 - [title]
```

**Note:** Do NOT continue to the next task. Stop is mandatory.

## Manual Override

The one-task-per-session rule is strict, but there are exceptions:

**User can continue if:**
- User explicitly requests continuation (their choice)
- Debugging in progress (need context for fix)

**Rotate immediately despite incomplete task:**
- 3+ consecutive failures
- Context feels wrong
- Major domain switch needed

**Note:** If user chooses to continue, they accept the risk of context degradation. The system should still output the stop recommendation.

## Anti-Patterns

### Don't Do This

| Anti-Pattern | Problem | Instead |
|--------------|---------|---------|
| Continuing after task complete | Context degrades, reviews skipped | STOP after each task |
| Rotating mid-task | Loses implementation context | Finish task, then rotate |
| No checkpoint before stop | Loses state | Always checkpoint first |
| Skipping code review | Issues compound across tasks | Review before commit |
| Fighting through rot | Compounds errors | Rotate and start fresh |

### Signs You're Fighting Context Rot

- "Let me try one more time..."
- "I know I can figure this out..."
- Making the same mistake repeatedly
- Edits getting longer and more complex
- User corrections increasing

**When you notice these:** Stop. Checkpoint. Rotate.

## Metrics for Self-Assessment

After completing a plan, review:

| Metric | Target | Red Flag |
|--------|--------|----------|
| Sessions per plan | 1 per task | Multiple tasks per session |
| Code reviews performed | 100% of tasks | Skipped reviews |
| Checkpoint success rate | 100% persisted | Failed checkpoints |
| User corrections | Rare | Frequent |
| Retry rate | < 20% of tasks | > 50% of tasks |

Track these to ensure the workflow is being followed.
