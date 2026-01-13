---
description: Remove Codex MCP server configuration
argument-hint: (no arguments)
---

# Uninstall Codex Command

Remove the Codex MCP server configuration from Claude Code settings.

## Usage

```
/uninstall-codex
```

## Instructions

When this command is invoked:

### 1. Read Existing Settings

Read `~/.claude/settings.json`.

**If file doesn't exist or has no codex entry:**
- Output: "Codex is not configured. Nothing to remove."
- STOP execution

### 2. Remove Codex Entry

Remove the `codex` entry from `mcpServers`:

**Before:**
```json
{
  "mcpServers": {
    "codex": {
      "type": "stdio",
      "command": "codex",
      "args": ["-m", "gpt-5.2-codex", "mcp-server"]
    },
    "other-server": { ... }
  }
}
```

**After:**
```json
{
  "mcpServers": {
    "other-server": { ... }
  }
}
```

**Important:** Preserve ALL other settings and MCP servers.

### 3. Write Updated Settings

Write the modified settings back to `~/.claude/settings.json`.

### 4. Verify Removal

```bash
cat ~/.claude/settings.json | grep -c '"codex"'
```

Expected: 0 (no matches)

**If verification fails:**
- Report error
- Show current settings content
- STOP

### 5. Output Summary

```
✓ Codex MCP server removed

Configuration updated: ~/.claude/settings.json

Note: Restart Claude Code for changes to take effect.

To re-enable Codex delegation, run: /setup-codex
```

## Notes

- This only removes the MCP server configuration
- Does NOT uninstall the Codex CLI (`npm uninstall -g @openai/codex`)
- Does NOT revoke OpenAI authentication
- Plans with `Dispatch: codex` will need to be updated or will fall back to alternative execution
