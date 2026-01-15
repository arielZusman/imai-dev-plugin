---
description: Enrich a Claude Code plan for standalone execution
argument-hint: [plan-path]
---

Use the Task tool to dispatch to the plan-enricher agent to enrich the specified plan.

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

## Task Scoping Validation

**"One Sentence Without 'And'" Rule:**

Each task MUST be describable in one sentence without conjoining unrelated work:
- ✓ "Update Angular core to v19" → properly scoped
- ✗ "Update Angular and fix lint errors and migrate SCSS" → should be 3 tasks

**Validation check:**
Review each task title. If it contains "and" connecting unrelated capabilities, the task is too broad.

```
For each task in plan:
  - Can it be described without "and"?
  - Does it have ONE clear outcome?
  - Will it result in ONE logical commit?

If NO to any: Break into separate tasks
```

This ensures each task = one focus = one commit = one review cycle.
