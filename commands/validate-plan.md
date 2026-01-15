---
description: Validate a Claude Code plan before implementation
argument-hint: [plan-path]
arguments:
  - name: plan_path
    description: Path to the plan file (optional - will search ~/.claude/plans/ if not provided)
    required: false
---

Use the plan-validator agent to validate the specified plan (or find and validate plans in ~/.claude/plans/).

If a plan path is provided: $ARGUMENTS.plan_path
Otherwise: Search for plans in ~/.claude/plans/ and ask user to select one.

The plan-validator agent will:
1. Load and parse the plan
2. Check architectural alignment with the codebase
3. Verify pattern consistency
4. **Validate task scoping** (see below)
5. **Validate enrichment standards** (for enriched plans - see below)
6. Identify risks and gaps
7. Generate actionable recommendations

## Task Scoping Validation

**"One Sentence Without 'And'" Rule:**

The validator MUST check that each task is properly scoped:

```
For each task:
  □ Can be described in one sentence without "and"
  □ Has ONE clear outcome
  □ Will result in ONE logical commit
  □ Doesn't combine unrelated work
```

**Examples:**
- ✓ "Update Angular core to v19" → properly scoped
- ✓ "Fix lint errors in auth module" → properly scoped
- ✗ "Update Angular and fix lint errors" → TOO BROAD
- ✗ "Add login and registration and password reset" → TOO BROAD

**If violations found:**
Report: "Task [N] violates scoping rule: '[title]' contains multiple unrelated concerns. Split into separate tasks."

This ensures each task = one focus = one commit = one review cycle.

## Enrichment Standards Validation

For enriched plans (in `docs/plans/`), the validator checks:

| Requirement | What's Validated |
|-------------|------------------|
| Fresh Session Entry Point | Reading order for session resume exists |
| Why field format | Business + Technical context, not generic |
| Time estimates | Per-task time estimates present |
| Context Verify notes | "What to check" for each required file |
| Rollback commands | Recovery path in Failure Modes |
| Task 0 | Prerequisite verification before Task 1 |
| Architecture | Written in prose, not bullets |
| Migration patterns | Before/after examples (upgrade plans only) |

These ensure plans are execution-ready for fresh Claude sessions with zero prior context.
