---
description: Prime Claude with project context at session start
---

# Session Context

## Current Branch
!git branch --show-current

## Uncommitted Changes
!git status --short

## Recent Commits
!git log --oneline -5

## Branch Work (commits since master)
!git log master..HEAD --oneline 2>/dev/null || echo "(on master)"

---

Ready. What would you like to work on?
