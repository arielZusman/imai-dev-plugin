# Codex Workflow Plugin (imai-dev)

This folder contains Codex custom prompts and Codex skills that mirror the
imai-dev workflow: brainstorm a plan, enrich it, validate it, and execute it
with checkpoints across fresh Codex sessions.

## Install

Custom prompts must live in your Codex home directory. Skills can be repo-scoped.

```bash
# Prompts (explicit slash commands)
mkdir -p ~/.codex/prompts
cp codex-plugin/prompts/*.md ~/.codex/prompts/

# Skills (repo-scoped)
mkdir -p .codex/skills
cp -R codex-plugin/skills/* .codex/skills/
```

Restart Codex after installing prompts or skills.

## Usage

Custom prompts are invoked as `/prompts:<name>` in Codex CLI or IDE.

Workflow:
1. Brainstorm a plan with Codex (freeform, or use your own prompt).
2. `/prompts:generate-plan PLAN_PATH=<path>`
3. `/prompts:validate-plan PLAN_PATH=<path>`
4. `/prompts:execute-plan PLAN=<plan-name-or-path>`
5. `/prompts:checkpoint PLAN_PATH=<path> TASK_NUMBER=<n> STATUS=<status>`

Notes:
- These prompts and skills assume plans live in `~/.codex/plans/` by default.
- Enriched plans are saved to `docs/plans/` in the repo, with checkpoints in
  `docs/plans/.state/`.
- Codex does not support sub-agents, so all execution is direct.
