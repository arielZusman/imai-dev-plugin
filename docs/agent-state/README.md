# Agent State Files

This directory stores persistent context to reduce session re-priming and context rot.

## Files
- `plan-summary.md`: Current plan index, current task, next actions.
- `task-template.md`: Template for per-task briefs.
- `task-<id>.md`: Per-task brief and handoff report.
- `session-log.md`: Rotating log of the last 20 session summaries.

## Conventions
- Keep per-task summaries under 12 lines.
- Use ISO timestamps (UTC) for log entries.
- Include a context budget for each task (tokens).
- Update `plan-summary.md` after every task completion.

## Rotation
- `session-log.md` should keep only the 20 most recent entries. Delete older entries when adding new ones.
