---
description: Enrich a Claude Code plan for standalone execution
argument-hint: [plan-path]
---

Use the Task tool to spawn the plan-enricher agent to enrich the specified plan.

If plan path provided: $ARGUMENTS
Otherwise: The agent will search ~/.claude/plans/ and prompt for selection.

The agent runs with fresh context and will:
1. Load and parse the original plan
2. Gather codebase context for referenced files
3. Enrich tasks with exact paths, verification, and review scope
4. Embed the mandatory execution workflow
5. Add per-task checklists to ensure code review is not skipped
6. Save to docs/plans/ with INDEX update
