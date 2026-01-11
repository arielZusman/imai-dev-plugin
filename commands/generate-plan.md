---
description: Enrich a Claude Code plan for standalone execution
argument-hint: [plan-path]
---

Use the Task tool to spawn the plan-enricher agent to enrich the specified plan.

If plan path provided: $ARGUMENTS
Otherwise: The agent will search ~/.claude/plans/ and prompt for selection.

The agent runs with fresh context and will:
1. Load and parse the original plan
2. Read TEMPLATE.md from skills/generate-implementation-plan/
3. Gather codebase context for referenced files
4. Enrich tasks with exact paths, verification, and review scope
5. Embed the mandatory execution workflow
6. Add per-task checklists (MUST include `/checkpoint` and `Review: /pr-review-toolkit:review-pr`)
7. Validate output before saving
8. Save to docs/plans/ with INDEX update

## Post-Enrichment Validation

After the agent completes, verify the enriched plan has:

```bash
# Check for mandatory checklist items in each task
grep -c "Review:.*pr-review-toolkit" docs/plans/*.md
grep -c "/checkpoint.*started" docs/plans/*.md
grep -c "/checkpoint.*completed" docs/plans/*.md
```

If any task is missing these elements, the enrichment failed. Re-run or manually fix.
