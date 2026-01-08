# Plan Execution Framework

A Claude Code plugin for structured plan creation, validation, and execution with checkpointing.

## Overview

This plugin provides a complete workflow for managing implementation plans:

```
/prime → /generate-plan → /validate-plan → /execute-plan → /checkpoint → /resume-plan
```

## Quick Start

```bash
# 1. Prime session with project context
/prime

# 2. Create a plan with Claude (plan mode)
"Help me implement feature X"
# Claude creates plan in ~/.claude/plans/

# 3. Enrich the plan for standalone execution
/generate-plan my-feature

# 4. Validate against codebase patterns
/validate-plan

# 5. Execute with orchestration
/execute-plan my-feature

# 6. When rotation is recommended (after 4 tasks or 30 min)
# Clear session, then:
/resume-plan my-feature
```

---

## Commands Reference

| Command | Description | Usage |
|---------|-------------|-------|
| `/prime` | Prime session with git context | `/prime` |
| `/prime-plan` | Load plan and task context | `/prime-plan [task-id]` |
| `/generate-plan` | Enrich plan for standalone execution | `/generate-plan [plan-path]` |
| `/validate-plan` | Validate plan against codebase | `/validate-plan [plan-path]` |
| `/execute-plan` | Execute plan with orchestration | `/execute-plan <plan> [tasks]` |
| `/checkpoint` | Save execution checkpoint | `/checkpoint <plan> <task> <status>` |
| `/resume-plan` | Resume plan from checkpoint | `/resume-plan <plan> [task-id]` |

**Note:** Each command file is self-contained with complete execution instructions. Claude Code reads the command file directly when invoked.

---

## Agents

| Agent | Purpose | Trigger |
|-------|---------|---------|
| `plan-enricher` | Enriches plans with inline context | Via `/generate-plan` |
| `plan-validator` | Validates plans against architecture | Via `/validate-plan` |

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
- Examples: `superpowers:test-driven-development`, `superpowers:systematic-debugging`

### Fork Mode
- `generate-implementation-plan` runs in isolated forked context
- Uses `Plan` agent type for enrichment
- Keeps main conversation clean during plan enrichment

---

## Rotation Heuristics

| Trigger | Action |
|---------|--------|
| 4+ tasks completed | Rotate after current task |
| 30+ minutes elapsed | Rotate after current task |
| 3+ consecutive failures | Rotate immediately |
| Context feels wrong | Rotate immediately |

---

## Dispatch Decisions (for /execute-plan)

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

## Directory Structure

```
.claude/
├── agents/
│   ├── plan-enricher.md
│   └── plan-validator.md
├── commands/
│   ├── checkpoint.md          # Self-contained execution instructions
│   ├── execute-plan.md        # Self-contained execution instructions
│   ├── generate-plan.md
│   ├── prime-plan.md
│   ├── prime.md
│   ├── resume-plan.md         # Self-contained execution instructions
│   └── validate-plan.md
├── skills/
│   ├── generate-implementation-plan/
│   ├── handoff-summary/
│   └── session-management/
├── settings.local.json
└── README.md
```
