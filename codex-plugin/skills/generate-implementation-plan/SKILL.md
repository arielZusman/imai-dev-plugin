---
name: generate-implementation-plan
description: Convert a draft plan into an execution-ready document for fresh Codex sessions; use after a plan is created or when preparing a plan for standalone execution with embedded context, verification, and checkpoints.
---

# Generate Implementation Plan (Codex)

Announce: "Using generate-implementation-plan to enrich this plan for standalone Codex execution."

## Quick Start

1. Identify the plan from ~/.codex/plans/ (fallback: ~/.claude/plans/)
2. Read and understand all tasks
3. Gather context for referenced files
4. Apply the template from assets/TEMPLATE.md
5. Save to docs/plans/YYYY-MM-DD-<feature-name>.md
6. Update docs/plans/INDEX.md

## Process

1) Identify the plan
- Ask which plan to use, or pick the most recent

2) Read the plan
- Understand scope, tasks, and dependencies

3) Gather context
- Read relevant files mentioned in the plan
- Verify paths exist before including them

4) Enrich
- Add a self-contained header and inline code context
- Add verification per task
- Add failure modes for non-trivial tasks

5) Save
- Write to docs/plans/YYYY-MM-DD-<feature-name>.md

6) Update index
- Add or update docs/plans/INDEX.md entry

## Key Principles

The enriched plan is for a fresh Codex session with zero prior context.

- Include all required context inline
- Be explicit about file paths and patterns
- Reference relevant code snippets directly in the plan
- Include verification criteria for every task
- Embed checkpoint workflow and one-task-per-session rule

## Skill Recommendations

For each task, recommend relevant skills that should be invoked. Use "none" when
no skill provides clear benefit. Keep recommendations minimal.

Example format:

**Recommended skill:** `session-management`
- Invoke BEFORE execution if session health is a concern

## Critical: Execution Workflow Embedding

The enriched plan MUST include these elements:

1) Execution Workflow section before tasks
2) Per-task checklist including checkpoint start/complete

Codex prompts are invoked as `/prompts:<name>`:
- Start checkpoint: `/prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=started`
- Complete checkpoint: `/prompts:checkpoint PLAN_PATH=<plan> TASK_NUMBER=<n> STATUS=completed`
- Execute next task: `/prompts:execute-plan PLAN=<plan-name>`

## References

- Template: assets/TEMPLATE.md
- Enrichment checklist: references/ENRICHMENT-CHECKLIST.md
- Execution guide: references/EXECUTION-GUIDE.md
- Failure modes: references/FAILURE-MODES-EXAMPLES.md

## Execution Handoff

After saving the enriched plan, offer:

Plan enriched and saved to docs/plans/<filename>.md. Ready to execute?
1) Execute now with /prompts:execute-plan
2) Open a fresh Codex session and run /prompts:execute-plan

## INDEX.md Maintenance

When updating docs/plans/INDEX.md:

- Add a row under Active Plans with:
  - Link to plan file
  - Created date (from filename)
  - Task count (count "### Task N:" headers)
  - Progress 0/N
  - Branch (from git context)
  - Status: Not Started

- If plan exists, update progress and status based on Execution Log
