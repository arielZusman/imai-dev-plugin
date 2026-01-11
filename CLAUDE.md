# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

imai-dev is a Claude Code plugin for structured plan creation, validation, and execution with checkpoint-based session management. It provides a complete workflow:

```
/prime → /generate-plan → /validate-plan → /execute-plan → /checkpoint → /resume-plan
```

## Plugin Architecture

### Components

| Type | Location | Purpose |
|------|----------|---------|
| Commands | `commands/*.md` | User-invocable slash commands |
| Agents | `agents/*.md` | Specialist sub-agents (plan-enricher, plan-validator) |
| Skills | `skills/*/SKILL.md` | Reusable workflows with supporting docs |
| Manifest | `.claude-plugin/plugin.json` | Plugin metadata |

### Core Workflow

1. **Plan Creation**: User creates plan in `~/.claude/plans/` (via Claude's plan mode)
2. **Enrichment**: `/generate-plan` adds inline context, verification steps, skill recommendations
3. **Validation**: `/validate-plan` checks against codebase patterns
4. **Execution**: `/execute-plan` orchestrates with sub-agent dispatch and checkpoints
5. **Persistence**: Checkpoints stored in `docs/plans/.state/`

### Dispatch Decisions (execute-plan)

Tasks are executed directly or via sub-agents based on complexity:
- **Direct**: Simple edits (< 50 lines), config changes
- **Sub-agent**: TDD tests, complex implementation, code review

### Session Rotation Heuristics

| Trigger | Action |
|---------|--------|
| 4+ tasks completed | Rotate after current task |
| 30+ minutes elapsed | Rotate after current task |
| 3+ consecutive failures | Rotate immediately |

## File Conventions

### Plan Files

- Raw plans: `~/.claude/plans/*.md`
- Enriched plans: `docs/plans/YYYY-MM-DD-<feature-name>.md`
- Checkpoints: `docs/plans/.state/<plan-slug>.checkpoint.md`
- Index: `docs/plans/INDEX.md`

### Skill Structure

Each skill in `skills/<name>/` contains:
- `SKILL.md` - Main skill definition with frontmatter
- Supporting docs (TEMPLATE.md, ENRICHMENT-CHECKLIST.md, etc.)

## Development Notes

### Adding Commands

Commands use YAML frontmatter for metadata:
```yaml
---
description: What it does
argument-hint: <required-arg> [optional]
arguments:
  - name: argname
    required: true/false
---
```

### Adding Skills

Skills use frontmatter to control execution context:
```yaml
---
name: skill-name
context: fork  # or inline
agent: Plan    # optional, agent type for fork mode
---
```

### Mandatory Review Workflow

The enriched plan template embeds `/pr-review-toolkit:review-pr` after each task. This is intentionally placed in per-task checklists, not just at the end, to prevent skipping.

## Dependencies

Works best with these optional plugins:
- **superpowers**: TDD, debugging, brainstorming skills
- **pr-review-toolkit**: Code review after each task
