---
name: handoff-summary
description: Generate a structured handoff for task or session transitions; use before clearing a session, after completing a task, or when context feels degraded.
---

# Handoff Summary Generator (Codex)

Announce: "Using handoff-summary to create a structured handoff for the next session/task."

## Handoff Structure

```markdown
## Handoff Summary

**Generated:** [timestamp]
**Plan:** [plan file path]
**Tasks Completed:** [list]

### What Was Done
- [Bullet list of completed work]
- [Include specific file changes]

### Current State
> Run these commands before filling in this section.
> ```bash
> npm run build && npm run test && git status
> ```

- **Build:** pass | fail (reason)
- **Tests:** X/Y passing (note expected failures)
- **Uncommitted Changes:** [list files or "none"]
- **Branch:** [branch name]

### Next Task
- **Task:** [number] - [title]
- **Complexity:** [simple/moderate/complex]
- **First Step:** [what to do first]

### Context Requirements
Files the next session MUST re-read:
- `path/to/file.ts` - [why]
- `path/to/other.ts` - [why]

### Decisions Made
| Decision | Rationale | Alternative Considered |
|----------|-----------|------------------------|
| [choice] | [why] | [what else] |

### Gotchas
- [Non-obvious issues]
- [Edge cases]

### Open Questions
- [ ] [Unresolved items]
```

## Integration with Checkpoint

When generating a handoff, also update the checkpoint file via:

/prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=completed

Provide the handoff notes when prompted.
