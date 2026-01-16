# imai-dev

A Claude Code plugin for structured plan creation, validation, and execution with checkpoint-based session management.

## Installation

### Via Marketplace
```
/plugin marketplace add <your-github-username>/imai-dev-plugin
```

### Direct Git Install
```
/plugin add github:<your-github-username>/imai-dev-plugin
```

### From Local Directory
```bash
claude --plugin-dir /path/to/imai-dev-plugin
```

## Overview

This plugin provides a complete workflow for managing implementation plans:

```
/prime → /generate-plan → /validate-plan → /execute-plan → /checkpoint
```

## Quick Start

```bash
# 1. Prime session with project context
/imai-dev:prime

# 2. Create a plan with Claude (plan mode)
"Help me implement feature X"
# Claude creates plan in ~/.claude/plans/

# 3. Enrich the plan for standalone execution
/imai-dev:generate-plan my-feature

# 4. Validate against codebase patterns
/imai-dev:validate-plan

# 5. Execute with orchestration
/imai-dev:execute-plan my-feature

# 6. When rotation is recommended, clear session and re-run:
/imai-dev:execute-plan my-feature
```

---

## Commands Reference

| Command | Description | Usage |
|---------|-------------|-------|
| `prime` | Prime session with git context | `/imai-dev:prime` |
| `generate-plan` | Enrich plan for standalone execution | `/imai-dev:generate-plan [plan-path]` |
| `validate-plan` | Validate plan against codebase | `/imai-dev:validate-plan [plan-path]` |
| `execute-plan` | Execute plan with orchestration | `/imai-dev:execute-plan <plan> [tasks]` |
| `checkpoint` | Save execution checkpoint | `/imai-dev:checkpoint <plan> <task> <status>` |

---

## Agents

| Agent | Purpose | Trigger |
|-------|---------|---------|
| `plan-enricher` | Enriches plans with inline context | Via `generate-plan` command |
| `plan-validator` | Validates plans against architecture | Via `validate-plan` command |

---

## Skills

| Skill | Purpose | Mode |
|-------|---------|------|
| `generate-implementation-plan` | Full plan enrichment workflow | Fork (isolated context) |
| `handoff-summary` | Structured handoff for session transitions | Inline |
| `session-management` | Session health and rotation guidelines | Inline |

---

## Key Features

### Checkpoint System
- Tracks task progress across sessions
- Stores handoff notes for context transfer
- Warns when rotation is recommended (4+ tasks or 30+ min)

### Execution Orchestration
- Dispatches simple tasks directly, complex tasks to sub-agents
- Supports parallel task execution
- Mandatory code review after each task

### Skill Recommendations
- Plans include recommended skills per task
- Checks all available skills (own + plugins + built-in)

### Fork Mode
- `generate-implementation-plan` runs in isolated forked context
- Uses `Plan` agent type for enrichment
- Keeps main conversation clean during plan enrichment

---

## Session Management

| Event | Action |
|-------|--------|
| Task completed | Checkpoint → Review → Commit → **STOP** |
| 3+ consecutive failures | Rotate immediately |

**Policy:** One task per session (automatic rotation). Session stops after every task completion.

---

## Dispatch Decisions (for execute-plan)

| Task Type | Execution | Rationale |
|-----------|-----------|-----------|
| Simple (< 50 lines) | Direct | Low overhead |
| Config changes | Direct | Trivial |
| TDD test writing | Sub-agent | Fresh context for test design |
| Complex implementation | Sub-agent | Context isolation |
| Code review | Sub-agent | Focused analysis |

---

## Checkpoint States

| Status | Meaning | Next Action |
|--------|---------|-------------|
| `started` | Beginning work on task | Implementation |
| `completed` | Task finished, verification passed | Next task |
| `error` | Error encountered | Fix and retry |
| `blocked` | Cannot proceed | Resolve blocker |

---

## Directory Conventions

This plugin expects the following directory structure in your project:

```
project/
├── docs/
│   └── plans/              # Enriched plans stored here
│       └── YYYY-MM-DD-feature/
│           ├── intro.md
│           ├── task-N.md
│           └── checkpoint.md
```

Plans are read from `~/.claude/plans/` and enriched versions are saved to `docs/plans/` as multi-file subdirectories.

---

## Dependencies

This plugin works best with the following optional plugins:

### superpowers (Recommended)
Provides skills referenced in plan task recommendations:
- `superpowers:test-driven-development`
- `superpowers:systematic-debugging`
- `superpowers:brainstorming`

### pr-review-toolkit (Recommended)
Provides code review capability used in the execution workflow:
- `/pr-review-toolkit:review-pr` - Called after each task completion

---

## Plugin Structure

```
imai-dev/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── plan-enricher.md
│   └── plan-validator.md
├── commands/
│   ├── checkpoint.md
│   ├── execute-plan.md
│   ├── generate-plan.md
│   ├── prime.md
│   └── validate-plan.md
├── skills/
│   ├── generate-implementation-plan/
│   ├── handoff-summary/
│   └── session-management/
└── README.md
```

---

## License

MIT
