# Codex Delegation Prompt Template

Use this template to build prompts for Codex task delegation. Codex is stateless - every delegation must include complete context.

---

## Template

```markdown
# Task: [TASK_TITLE]

## Context
- **Project:** [REPO_NAME]
- **Working Directory:** [PWD]
- **Task:** [TASK_NUMBER] of [TOTAL_TASKS] in plan [PLAN_NAME]
- **Complexity:** [COMPLEXITY_INDICATOR]

## Objective
[One sentence describing what this task accomplishes]

## Files to Modify

| File | Action | Purpose |
|------|--------|---------|
| [FILE_PATH] | create/modify | [why] |

## Current Code State

### File: [FILE_PATH_1]
```[language]
[CURRENT_FILE_CONTENT]
```

### File: [FILE_PATH_2]
```[language]
[CURRENT_FILE_CONTENT]
```

## Task Steps

[COPY_TASK_STEPS_FROM_PLAN]

## Constraints

- Modify ONLY the files listed above
- Follow existing code patterns in the project
- Do NOT add new dependencies unless explicitly stated
- Do NOT refactor unrelated code
- Preserve all existing functionality

## Code Patterns to Follow

[INCLUDE_RELEVANT_PATTERNS_FROM_CODEBASE]

## Expected Output

Return modified code for EACH file in this exact format:

### File: path/to/file.ts
```typescript
// Complete file content here
// Include ALL code, not just changes
```

### File: path/to/another-file.ts
```typescript
// Complete file content here
```

## Verification

After implementation, these checks should pass:
[COPY_VERIFICATION_STEPS_FROM_PLAN]
```

---

## Field Descriptions

| Field | Source | Notes |
|-------|--------|-------|
| TASK_TITLE | Plan task title | |
| REPO_NAME | `basename $(git rev-parse --show-toplevel)` | |
| PWD | Current working directory | |
| TASK_NUMBER | From checkpoint or plan position | |
| TOTAL_TASKS | Count from plan | |
| PLAN_NAME | Plan filename without path | |
| COMPLEXITY_INDICATOR | Plan task complexity (🟢/🟡/🔴) | |
| FILE_PATH | From task's Context Requirements | Only REQUIRED files |
| CURRENT_FILE_CONTENT | Read file contents | Codex is stateless |
| TASK_STEPS | Copy from plan task | Verbatim |
| CODE_PATTERNS | Grep similar implementations | Help maintain consistency |
| VERIFICATION_STEPS | Copy from plan task's Verify section | |

## Building the Prompt

1. **Read task from plan** - Get title, steps, verify section, context requirements
2. **Read all required files** - Codex needs full file contents
3. **Find code patterns** - Grep for similar implementations to include as examples
4. **Populate template** - Fill in all fields
5. **Validate completeness** - Ensure no [PLACEHOLDER] fields remain

## Parsing Codex Response

1. **Extract file blocks** - Look for `### File:` headers
2. **Parse code content** - Extract content between triple backticks
3. **Validate paths** - Ensure paths match expected files
4. **Apply changes** - Use Write tool for each file
5. **Verify** - Run task verification steps

## Error Handling

If Codex response is malformed or incomplete:
- Log error details
- Ask user via AskUserQuestion:
  - Retry Codex delegation
  - Fall back to Claude direct execution
  - Mark task as blocked
