---
description: Enrich a Claude Code plan for Codex CLI execution
argument-hint: [plan-path]
---

Use the Task tool to dispatch to the plan-enricher agent to enrich the specified plan for Codex CLI execution.

If plan path provided: $ARGUMENTS
Otherwise: The agent will search ~/.claude/plans/ and ~/.codex/plans/ and prompt for selection.

The agent runs with fresh context and will:
1. Load and parse the original plan
2. Read INTRO-TEMPLATE.md and TASK-TEMPLATE.md from skills/generate-codex-plan/
3. Gather codebase context for referenced files
4. Enrich tasks with exact paths, verification, and review scope
5. Embed the mandatory execution workflow (Codex + Claude Code split)
6. Add per-task checklists (MUST include `/checkpoint` and `Review: /pr-review-toolkit:review-pr`)
7. Remove sub-agent and skill recommendations (Codex doesn't support these)
8. Use multi-file format (always, for automation support)
9. Validate output before saving
10. Save to docs/plans/codex/ with INDEX update

## Post-Enrichment Validation

After the agent completes, verify the enriched plan has:

```bash
# Check for mandatory checklist items in each task
grep -c "Review:.*pr-review-toolkit" docs/plans/codex/**/*.md
grep -c "/checkpoint.*started" docs/plans/codex/**/*.md
grep -c "/checkpoint.*completed" docs/plans/codex/**/*.md

# Verify Codex-specific elements
grep -c "Codex CLI" docs/plans/codex/**/*.md
grep -c "Claude Code" docs/plans/codex/**/*.md
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

## Key Differences from Regular Plans

**For Codex CLI plans:**
- No sub-agent dispatch (removed)
- No skill recommendations (removed)
- Always multi-file format (even small plans)
- Clear Codex vs Claude Code responsibility split
- Dual checklists (Codex tasks + Claude Code tasks)
- Output to `docs/plans/codex/` directory

## Usage

```bash
# Generate Codex plan from existing plan
/generate-codex-plan ~/.claude/plans/my-feature.md

# Interactive selection
/generate-codex-plan

# From codex plans directory
/generate-codex-plan ~/.codex/plans/my-feature.md
```

## After Generation

The enriched plan will be saved to:
- `docs/plans/codex/YYYY-MM-DD-<feature-name>/intro.md`
- `docs/plans/codex/YYYY-MM-DD-<feature-name>/task-N.md` (one per task)

Each task file can be passed to Codex CLI individually, enabling automation scripts to iterate through tasks.
