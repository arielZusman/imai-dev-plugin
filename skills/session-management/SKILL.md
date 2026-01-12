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

## Rotation Triggers

### Hard Triggers (Always Rotate)

| Trigger | Detection | Action |
|---------|-----------|--------|
| **4+ tasks completed** | Checkpoint counter | Rotate after current task |
| **30+ minutes elapsed** | Session start time | Rotate after current task |
| **Major error recovery** | 3+ failed attempts | Rotate immediately |
| **Context feels wrong** | Repeating yourself, wrong assumptions | Rotate immediately |

### Soft Triggers (Consider Rotating)

| Trigger | Detection | Action |
|---------|-----------|--------|
| **2-3 tasks completed** | Checkpoint counter | Continue if simple tasks ahead |
| **15-30 minutes elapsed** | Session start time | Continue if nearly done |
| **One complex task done** | 🔴 complexity completed | Evaluate remaining work |
| **Switching domains** | Frontend → Backend, etc. | Rotate for fresh context |

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
   /resume-plan <plan-path>
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
    ├─ Task completed?
    │   │
    │   ├─ Tasks this session >= 4? ──→ ROTATE
    │   │
    │   ├─ Time elapsed > 30 min? ──→ ROTATE
    │   │
    │   ├─ Tasks this session = 3?
    │   │   │
    │   │   ├─ Next task is 🔴 complex? ──→ ROTATE
    │   │   └─ Next task is 🟢 simple? ──→ CONTINUE
    │   │
    │   └─ Tasks this session < 3? ──→ CONTINUE
    │
    ├─ Error occurred?
    │   │
    │   ├─ 3+ consecutive failures? ──→ ROTATE
    │   └─ Recoverable error? ──→ CONTINUE (checkpoint error)
    │
    └─ User reports confusion? ──→ ROTATE
```

## Task Budgeting

When starting a plan, estimate tasks per session:

| Task Complexity | Tasks Per Session |
|-----------------|-------------------|
| All 🟢 Simple | 4-5 tasks |
| Mixed 🟢/🟡 | 3-4 tasks |
| Includes 🔴 Complex | 2-3 tasks |
| All 🔴 Complex | 1-2 tasks |

**Planning example:**
```
Plan: 12 tasks (4 🟢, 6 🟡, 2 🔴)

Estimated sessions needed:
- Session 1: Tasks 1-4 (🟢🟢🟡🟡) = 4 tasks
- Session 2: Tasks 5-7 (🟡🟡🔴) = 3 tasks
- Session 3: Tasks 8-10 (🟡🟡🔴) = 3 tasks
- Session 4: Tasks 11-12 (🟢🟢) + final review = 2 tasks

Total: ~4 sessions for 12 tasks
```

## Integration with Commands

### /checkpoint

Reports rotation recommendation:
```
Checkpoint saved.
Tasks this session: 3
⚠️ Consider rotating after next task (approaching limit).
```

### /resume-plan

Resets session counter:
```
Session started fresh.
Tasks this session: 0
Rotation recommended after: 4 tasks or 30 min
```

### /execute-plan

Pauses for rotation:
```
Task 4 complete.
⚠️ Rotation recommended: 4 tasks completed.

Options:
1. Pause and rotate (recommended)
2. Continue (not recommended - quality may degrade)
```

## Manual Override

Sometimes you should ignore rotation recommendations:

**Continue despite trigger:**
- Only 1 trivial task remaining
- In middle of debugging (need context)
- User explicitly requests continuation

**Rotate despite no trigger:**
- Major domain switch (frontend → backend)
- Significant error recovery just completed
- Starting complex task after simple ones
- Session "feels" degraded

## Anti-Patterns

### Don't Do This

| Anti-Pattern | Problem | Instead |
|--------------|---------|---------|
| Ignoring rotation warnings | Quality degrades silently | Respect the heuristics |
| Rotating mid-task | Loses implementation context | Finish task, then rotate |
| No checkpoint before rotate | Loses state | Always checkpoint first |
| Rotating after every task | Excessive overhead | Trust the heuristics |
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
| Sessions per plan | 3-5 for 10-task plan | 8+ sessions |
| Tasks per session | 3-4 average | < 2 average |
| Rotation triggers hit | Planned rotations | Emergency rotations |
| User corrections | Rare | Frequent |
| Retry rate | < 20% of tasks | > 50% of tasks |

Track these to calibrate your rotation heuristics over time.
