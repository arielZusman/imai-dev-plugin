# Execution Guide (Codex)

How to execute an enriched implementation plan in Codex.

## Resuming a Plan

Any session can resume an enriched plan by:
1) Reading the plan file from docs/plans/
2) Scanning the Execution Log for current position
3) Following the embedded Execution Workflow for each task

## Related Skills

Invoke when relevant during execution:
- generate-implementation-plan (plan enrichment)
- session-management (rotation and session health)
- handoff-summary (handoff notes and checkpoint updates)

## Reference: Per-Task Cycle

The embedded Execution Workflow mandates this cycle for every task:

1) **Start:** /prompts:checkpoint STATUS=started
2) **Implement:** Complete task steps
3) **Verify:** Run the task's verification step
4) **Review:** git diff --staged and git status
5) **Fix if needed:** Address critical issues (max 2 cycles)
6) **Commit:** After review passes
7) **Complete:** /prompts:checkpoint STATUS=completed

One task per session. Stop after each completion and resume with
/prompts:execute-plan in a fresh session.
