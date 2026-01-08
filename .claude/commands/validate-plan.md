---
description: Validate a Claude Code plan before implementation
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
4. Identify risks and gaps
5. Generate actionable recommendations
