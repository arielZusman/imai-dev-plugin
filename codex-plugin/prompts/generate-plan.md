---
description: Enrich a draft plan for standalone execution across Codex sessions
argument-hint: [ PLAN_PATH=<path> ]
---

Use the $generate-implementation-plan skill to enrich a plan for standalone execution.

If PLAN_PATH is provided, use it. Otherwise:
1) Look for plans in ~/.codex/plans/
2) If none found, check ~/.claude/plans/ for legacy plans
3) Ask the user to pick a plan

If the skill is unavailable, follow the process in
`.codex/skills/generate-implementation-plan/SKILL.md` after installing the skill
from codex-plugin/skills.
