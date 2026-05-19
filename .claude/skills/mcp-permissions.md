# MCP Server Permissions

Quick reference for setting up MCP server permissions in Claude Code.

## Permission Format

Add to `.claude/settings.local.json` in the `permissions.allow` array:

```json
"mcp__<SERVER-NAME>"
```

## Examples

- Playwright: `"mcp__playwright"`
- Filesystem: `"mcp__filesystem"`  
- Git: `"mcp__git"`

## Full Example

```json
{
  "permissions": {
    "allow": [
      "mcp__playwright",
      "mcp__filesystem"
    ]
  }
}
```

This auto-approves all tool calls from these MCP servers without permission prompts.

## Why It Works

MCP tools are named: `mcp__<SERVER-NAME>__<TOOL-NAME>`

Permission rules match by prefix, so `"mcp__playwright"` matches all playwright tools.
