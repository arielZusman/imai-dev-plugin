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
5. Identify risks and gaps
6. Generate actionable recommendations

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
