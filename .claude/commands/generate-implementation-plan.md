---
description: Generate an enriched implementation plan with task packets
---

# INPUTS
This command requires a source plan or spec path via `$ARGUMENTS`.
If `$ARGUMENTS` is empty, ask for the path and stop.

# OUTPUT
Write the enriched plan to `docs/plans/YYYY-MM-DD-<topic>.md` unless a specific output path is included in `$ARGUMENTS`.

# PROCESS
1. Read the source file.
2. Produce an enriched plan with the required header format.
3. Decompose into tasks with explicit dependencies and context budgets.
4. Include verification commands and expected results for each task.
5. Add a short success metrics section at the end.

# REQUIRED HEADER
```
# <Feature Name> Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** <one sentence>

**Architecture:** <2-3 sentences>

**Tech Stack:** <key technologies>

---
```

# TASK PACKET TEMPLATE
```
### Task N: <Title>

**Files:**
- Create: `path`
- Modify: `path:line`
- Test: `path`

**Context Budget:** <tokens>
**Depends On:** <task ids>
**Definition of Done:** <short checklist>

**Step 1: Write the failing test**
<test code>

**Step 2: Run test to verify it fails**
Run: <command>
Expected: FAIL with <message>

**Step 3: Write minimal implementation**
<code>

**Step 4: Run test to verify it passes**
Run: <command>
Expected: PASS

**Step 5: Commit**
<git add + git commit>
```

# FOOTER
Include:
- Task execution order
- Quick wins vs. larger refactors
- Success metrics
