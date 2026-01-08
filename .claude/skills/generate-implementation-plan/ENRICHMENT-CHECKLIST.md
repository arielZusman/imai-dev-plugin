# Enrichment Checklist

Verify each item before saving the enriched plan.

## Required Content

- [ ] **Project context** - Repo, service, branch
- [ ] **Goal** - One sentence summary
- [ ] **Architecture** - Design approach
- [ ] **Execution Workflow section** - At TOP (after Architecture, BEFORE tasks) with mandatory per-task cycle
- [ ] **Per-task checklist** - Each task ends with checklist including review step
- [ ] **Relevant code context** - Key snippets the executing Claude needs
- [ ] **Exact file paths** - With line numbers for modifications
- [ ] **Clear intent or exact content** - Intent for logic, exact content for templates/config
- [ ] **Step granularity** - Each step is ONE action (2-5 min), not a bundle of actions
- [ ] **Commands with expected output** - What to run, what specific output to expect (e.g., "Expected: FAIL with 'Cannot find name'")
- [ ] **Verification per task** - How to confirm completion
- [ ] **Task dependencies** - "Depends on" field for each task
- [ ] **Parallel groups** - Independent tasks grouped for parallel execution
- [ ] **Complexity indicators** - Each task marked 🟢/🟡/🔴
- [ ] **Agent recommendations** - Per-task agent when beneficial (use `—` if none helps)
- [ ] **Final checklist** - Tests, build, feature verification
- [ ] **Gotchas** - Non-obvious issues
- [ ] **Review protocol** - Per-task review scope, final review step
- [ ] **Execution Log** - Table with row per task, all marked ⏳ Pending

## Content Guidelines

**Include exact content when:**
- Translation keys (exact strings matter)
- HTML templates (structure must be exact)
- Config values (specific values required)
- Simple boilerplate (pattern is exact copy)

**Use intent + location when:**
- Business logic (Claude writes better code seeing actual context)
- Complex functions (needs adaptation to current code)
- Conditional logic (context-dependent)

**Rule of thumb:** Include exact content if the executing Claude needs the *exact characters*. Use intent if Claude can derive it from context.
