## Checklist

**For Codex CLI:**
- [ ] Context requirements read
- [ ] All code changes implemented per steps
- [ ] Files created/modified as specified

**For Claude Code:**
- [ ] `/checkpoint {{PLAN_PATH}} {{TASK_NUMBER}} started`
- [ ] Verification passed
- [ ] Build passes: `npm run build`
- [ ] Tests pass: `npm run test`
- [ ] Review: `/pr-review-toolkit:review-pr staged {{ASPECTS}}`
- [ ] Committed (format: `{{TYPE}}: {{TASK_TITLE}} (Task {{TASK_NUMBER}})`)
- [ ] `/checkpoint {{PLAN_PATH}} {{TASK_NUMBER}} completed`
- [ ] **STOP** - Session pauses here
