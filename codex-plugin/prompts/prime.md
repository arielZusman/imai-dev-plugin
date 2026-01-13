---
description: Prime Codex with repo context (branch, status, recent commits)
---

Run the following commands and summarize the results:

1) git branch --show-current
2) git status --short
3) git log --oneline -5
4) git log master..HEAD --oneline 2>/dev/null || echo "(on master)"

Then ask: "Ready. What would you like to work on?"
