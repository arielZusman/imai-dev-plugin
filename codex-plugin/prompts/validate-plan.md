---
description: Validate an enriched plan against codebase patterns
argument-hint: [ PLAN_PATH=<path> ]
---

Validate the specified plan (PLAN_PATH) or, if not provided, prompt the user
to choose a plan from docs/plans/ (or ~/.codex/plans/ for draft plans).

Validation must include:
- Scope: affected files and modules are explicit
- Current state: what problem exists now
- Desired state: what should change
- Success criteria: how to verify each task
- Task scoping: each task must be describable in one sentence without "and"
- Pattern consistency: compare against existing code patterns
- Risk assessment: critical issues, important concerns, suggestions

Output a structured validation report with actionable fixes.
