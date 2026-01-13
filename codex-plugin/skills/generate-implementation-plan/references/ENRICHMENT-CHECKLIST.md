# Enrichment Checklist (Codex)

Verify each item before saving the enriched plan.

## Required Content

### Plan Structure
- [ ] **Project context** - Repo, service, branch
- [ ] **Goal** - One sentence summary
- [ ] **Architecture** - Design approach
- [ ] **Execution Workflow section** - At TOP (after Architecture, BEFORE tasks)
- [ ] **Relevant code context** - Key snippets the executor needs

### Per-Task Requirements
- [ ] **Exact file paths** - With line numbers for modifications
- [ ] **Context Requirements** - REQUIRED vs REFERENCE files per task
- [ ] **Clear intent or exact content** - Intent for logic, exact content for templates/config
- [ ] **Step granularity** - Each step is ONE action (2-5 min)
- [ ] **Commands with expected output** - What to run, what to expect
- [ ] **Verification per task** - How to confirm completion
- [ ] **Failure modes** - "If X happens, likely cause is Y, fix by Z"
- [ ] **Handoff notes placeholder**
- [ ] **Per-task checklist** - Including checkpoint start/complete calls

### Task Coordination
- [ ] **Task dependencies** - "Depends on" field for each task
- [ ] **Complexity indicators** - Each task marked Simple/Moderate/Complex
- [ ] **Dispatch hints** - direct only (Codex has no sub-agents)
- [ ] **Skill recommendations** - Use "none" if none helps

### Plan-Level
- [ ] **Final checklist** - Tests, build, feature verification
- [ ] **Gotchas** - Non-obvious issues
- [ ] **Execution Log** - Table with row per task, all marked Pending

## Context Requirements

Each task should specify files the executor MUST re-read before starting.

**Format:**
```markdown
**Context Requirements:**
- **Required** (must re-read before starting):
  - `src/services/auth.service.ts` - sections: validateToken, refreshToken
  - `src/types/auth.types.ts` - all
- **Reference** (consult if needed):
  - `docs/auth-flow.md` - sections: Token Lifecycle
```

## Failure Modes

Document common failure patterns to help the executor self-diagnose.

**Format:**
```markdown
**Failure Modes:**
- **If build fails with "Cannot find module":** Likely missing import. Check imports at top of file.
- **If tests fail with "timeout":** Async operation not awaited. Check for missing `await`.
```
