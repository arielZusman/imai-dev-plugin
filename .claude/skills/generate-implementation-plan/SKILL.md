---
name: generate-implementation-plan
description: |
  Converts basic plans into execution-ready documents with embedded context,
  verification steps, and mandatory code review workflow.

  Use after creating a plan, or when preparing plans for fresh sessions
  where inline context is essential for execution.
context: fork
agent: Plan
---

# Generate Implementation Plan

**Announce:** "I'm using the generate-implementation-plan skill to enrich this plan for standalone execution. This will embed the mandatory execution workflow and per-task checklists."

## Quick Start

1. Identify plan from `~/.claude/plans/`
2. Read and gather context for referenced files
3. Apply template from [TEMPLATE.md](TEMPLATE.md)
4. Save to `docs/plans/YYYY-MM-DD-<feature-name>.md`
5. Update `docs/plans/INDEX.md`

## Process

1. **Identify the plan** - Ask user which plan or use the most recent
2. **Read the plan** - Understand tasks and scope
3. **Gather context** - Read relevant files mentioned in the plan
4. **Enrich** - Add self-contained header and inline code context
5. **Add verification** - How to confirm each task is complete
6. **Add skill recommendations** - Which skills to invoke per task
7. **Save** - Write to `docs/plans/YYYY-MM-DD-<feature-name>.md`
8. **Update INDEX** - Add/update entry in `docs/plans/INDEX.md`

## Key Principles

The enriched plan is for a **fresh Claude session** with zero prior context.

- Include all context inline (no memory of your conversation)
- Be explicit about file locations and patterns
- Reference relevant code snippets directly in the plan
- Intent for business logic, exact content for templates/config
- Verification criteria for every task
- Skill recommendations for specialized workflows

## Skill Recommendations

For each task, recommend relevant skills that should be invoked. Check all available skills including:
- Built-in skills (e.g., from superpowers plugin)
- Project-specific skills (in `.claude/skills/`)
- Third-party plugin skills

| Task Type | Recommended Skill | When to Use |
|-----------|------------------|-------------|
| Writing tests first | `superpowers:test-driven-development` | TDD workflow, test before implementation |
| Debugging issues | `superpowers:systematic-debugging` | Bug fixes, unexpected behavior |
| Multiple independent tasks | `superpowers:dispatching-parallel-agents` | 2+ tasks can run in parallel |
| Planning implementation | `superpowers:writing-plans` | Multi-step feature planning |
| Code review | `superpowers:requesting-code-review` | After completing tasks |
| Session boundaries | `handoff-summary` | Before rotating sessions |
| Session health | `session-management` | Context degradation suspected |

**Per-task format:**
```markdown
**Recommended skill:** `superpowers:test-driven-development`
  - Invoke BEFORE writing implementation code
  - Ensures tests are written first
```

**Selection criteria:**
- Match task type to skill purpose
- Use `—` if no skill provides clear benefit
- Include invocation timing (before/during/after implementation)

## Agent Recommendations

When a specialist agent would benefit a task, recommend it. Common patterns:

| Task Type | Agent Type | When to Use |
|-----------|------------|-------------|
| Test writing | `test-writer` | TDD test design needs fresh context |
| Complex implementation | `general-purpose` | Context isolation benefit |
| Code exploration | `Explore` | Understanding unfamiliar code |
| Architecture decisions | `Plan` | Design decisions, trade-offs |

**Selection criteria:**
- Match task domain to agent specialty
- Use `—` if no agent provides clear benefit
- Consider project-specific agents if available

## Critical: Execution Workflow Embedding

The enriched plan MUST include these elements to ensure code reviews are not skipped:

1. **Execution Workflow section** - Placed BEFORE tasks, not buried at the end
2. **Per-task checklist** - Each task ends with a checklist including the review step

This workflow mandates:
- Code review for EVERY task (`/pr-review-toolkit:review-pr`)
- Status updates in Execution Log
- Fix-and-retry cycle for critical issues (max 2 cycles)
- Session handoff protocol

Without these, executors will skip reviews because the instructions are too far from where they're working.

## References

- **Plan template**: [TEMPLATE.md](TEMPLATE.md)
- **Enrichment checklist**: [ENRICHMENT-CHECKLIST.md](ENRICHMENT-CHECKLIST.md)
- **Failure modes examples**: [FAILURE-MODES-EXAMPLES.md](FAILURE-MODES-EXAMPLES.md)
- **Execution guide**: [EXECUTION-GUIDE.md](EXECUTION-GUIDE.md)

## Execution Handoff

After saving the enriched plan, offer:

> **Plan enriched and saved to `docs/plans/<filename>.md`. Ready to execute?**
>
> 1. **Execute now** - Start working through tasks in this session
> 2. **New session** - Open fresh session with just the plan file

## INDEX.md Maintenance

After saving an enriched plan, update `docs/plans/INDEX.md`:

1. Read current INDEX.md
2. Check if plan already exists in table
3. If new: Add row to "Active Plans" with:
   - Link to plan file
   - Created date (from filename)
   - Task count (count `### Task N:` headers)
   - Progress: 0/N
   - Branch (from plan's Git Context)
   - Status: ⏳ Not Started
4. If existing: Update progress and status based on Execution Log
5. Write updated INDEX.md

When a plan reaches 100% completion:
- Move from "Active Plans" to "Completed Plans"
- Add completion date
