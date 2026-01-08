---
description: Resume execution of an enriched plan
arguments:
  - name: plan
    description: Plan filename (e.g., "influencer-tier-filter" or full path)
    required: true
---

Resume execution of an enriched implementation plan.

## Step 1: Load the Plan

Read the specified plan file:
- If argument is a filename: search `docs/plans/*$ARGUMENTS.plan*.md`
- If argument is a path: use directly

## Step 2: Find Current Position

Scan the Execution Log table at the bottom of the plan:
1. Find the first task with status ⏳ Pending or 🔄 In Progress
2. If all tasks are ✅ Done, check if Final Review is complete
3. If everything is done, report plan completion

## Step 3: Execute

Follow the **Execution Workflow** section embedded in the plan. For each task:
1. Update status to 🔄 In Progress
2. Implement the task steps
3. Run the task's Verify step
4. Run code review per the plan's Review Protocol
5. Commit after review passes
6. Update status to ✅ Done

The plan is self-contained - all instructions are embedded.
