# Execution Guide

How to execute an enriched implementation plan.

> **Note:** The execution workflow is now embedded directly in each enriched plan under the "Execution Workflow" section. This guide provides supplementary information.

## Resuming a Plan

Any session can resume an enriched plan by:
1. Reading the plan file from `docs/plans/`
2. Scanning the Execution Log for current position (first non-✅ task)
3. Following the embedded Execution Workflow for each task

## Related Skills

Invoke when relevant during execution:
- `brainstorming` - Before writing new functionality
- `test-driven-development` - When implementing features
- `systematic-debugging` - When encountering bugs
- `verification-before-completion` - Before claiming done

## Reference: Per-Task Cycle

The embedded Execution Workflow mandates this cycle for every task:

1. **Start:** Update status to 🔄 In Progress
2. **Implement:** Complete task steps
3. **Verify:** Run task's verification step
4. **Review:** Run `/pr-review-toolkit:review-pr` with matching aspects
5. **Fix if needed:** Address critical issues (max 2 cycles)
6. **Commit:** After review passes
7. **Complete:** Update status to ✅ Done
