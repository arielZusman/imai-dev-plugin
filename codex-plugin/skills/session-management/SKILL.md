---
name: session-management
description: Evaluate session health and rotation for Codex; use when context feels degraded, after completing a task, or before long execution sequences.
---

# Session Management & Rotation (Codex)

Announce: "Using session-management to evaluate session health and rotation needs."

## One Task Per Session

Stop after EACH task. This is mandatory.

Reasons:
- Fresh context for each task
- Checkpoints remain current
- Reduces compound errors

## Rotation Protocol

Before rotating:
1) Save checkpoint:
   /prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=completed
2) Verify state:
   npm run build
   npm run test
   git status
3) Document position (task number, phase, blockers)

After rotating:
- Start a fresh Codex session
- Run /prompts:execute-plan PLAN=<plan-name>

## Signs of Context Rot

- Repeating the same mistakes
- Forgetting recently read files
- Re-implementing existing logic
- Confusion about task status

If you see these, stop, checkpoint, and rotate.
