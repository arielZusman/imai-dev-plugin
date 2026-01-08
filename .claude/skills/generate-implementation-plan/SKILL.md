---
name: generate-implementation-plan
description: Enriches Claude Code plans with full context for standalone execution. Use after plan creation, when generating implementation plans, or when preparing plans for fresh sessions.
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
6. **Save** - Write to `docs/plans/YYYY-MM-DD-<feature-name>.md`
7. **Update INDEX** - Add/update entry in `docs/plans/INDEX.md`

## Key Principles

The enriched plan is for a **fresh Claude session** with zero prior context.

- Include all context inline (no memory of your conversation)
- Be explicit about file locations and patterns
- Reference relevant code snippets directly in the plan
- Intent for business logic, exact content for templates/config
- Verification criteria for every task

## Agent Recommendations

For Discovery-imai projects, recommend a subagent per task when it benefits implementation. Only include if the agent's specialty matches the task.

| Agent | Domain | Use When |
|-------|--------|----------|
| `imai-frontend:angular-expert` | Frontend | Angular components, RxJS, reactive forms |
| `imai-frontend:ui-ux-expert` | Frontend | Figma→Angular, design system, accessibility |
| `imai-frontend:translation-expert` | Frontend | i18n, hardcoded strings, translations |
| `imai-backend:api-expert` | Backend | REST endpoints, auth, API docs |
| `imai-backend:database-expert` | Backend | Schema design, query optimization, migrations |
| `imai-backend:nestjs-expert` | Backend | NestJS modules, entities, microservices |
| `imai-qa:qa-performance-tester` | QA | Load testing, bottleneck analysis |
| `imai-qa:qa-security-tester` | QA | Security validation, OWASP compliance |
| `imai-qa:qa-integration-tester` | QA | Cross-service testing, API contracts |
| `imai-qa:performance-optimizer` | QA | Query optimization, caching, bundle size |
| `imai-qa:bug-hunter` | QA | Production issues, root cause analysis |
| `imai-qa:qa-regression-tester` | QA | Backward compatibility, pre-release |
| `imai-qa:qa-documentation-manager` | QA | Docs, changelogs, PR descriptions |

**Selection criteria:**
- Match task domain to agent specialty
- Use `—` if no agent provides clear benefit
- Prefer specific agents over general ones (e.g., `database-expert` for migrations, not `nestjs-expert`)

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
