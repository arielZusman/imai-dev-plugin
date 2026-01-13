---
description: Configure Codex MCP server for task delegation
argument-hint: (no arguments)
---

# Setup Codex Command

Configure OpenAI's Codex CLI as an MCP server for delegating task execution.

## Usage

```
/setup-codex
```

## Prerequisites

- Codex CLI installed globally: `npm install -g @openai/codex`
- OpenAI authentication: `codex login`

## Instructions

When this command is invoked:

### 1. Check Codex CLI Installation

```bash
codex --version
```

**If command fails:**
- Output: "Codex CLI not found. Install it with: `npm install -g @openai/codex`"
- STOP execution

### 2. Read Existing Settings

Read `~/.claude/settings.json` if it exists.

**Important:** Preserve all existing settings. We are MERGING, not replacing.

### 3. Add Codex MCP Server Configuration

Add the following to `mcpServers`:

```json
{
  "mcpServers": {
    "codex": {
      "type": "stdio",
      "command": "codex",
      "args": ["-m", "gpt-5.2-codex", "mcp-server"]
    }
  }
}
```

**Merge strategy:**
- If `mcpServers` doesn't exist, create it
- If `codex` entry already exists, update it
- Preserve all other MCP servers

### 4. Write Updated Settings

Write the merged settings to `~/.claude/settings.json`.

### 5. Verify Configuration

```bash
cat ~/.claude/settings.json | grep -A5 '"codex"'
```

**If verification fails:**
- Report error
- Retry write once
- If still fails: STOP and report to user

### 6. Check Authentication Status

Attempt to verify Codex authentication (if possible).

**If not authenticated:**
- Output: "Codex configured but not authenticated. Run `codex login` to authenticate."

### 7. Output Summary

```
✓ Codex MCP server configured

Configuration:
- Settings file: ~/.claude/settings.json
- Model: gpt-5.2-codex
- Type: stdio

Next steps:
1. If not authenticated, run: codex login
2. Use `Dispatch: codex` in plan tasks to delegate to Codex
3. Run /uninstall-codex to remove configuration

Note: Restart Claude Code for MCP changes to take effect.
```

## Example MCP Tool Usage

After setup, Codex is available via MCP tool:

```
mcp__codex__codex
```

The execute-plan command will automatically use this when tasks have `Dispatch: codex`.

## Troubleshooting

**"Codex CLI not found"**
- Install: `npm install -g @openai/codex`
- Verify PATH includes npm global bin directory

**"Authentication required"**
- Run: `codex login`
- Follow OAuth flow to authenticate with OpenAI

**"MCP server not responding"**
- Restart Claude Code
- Verify Codex works standalone: `codex --help`
