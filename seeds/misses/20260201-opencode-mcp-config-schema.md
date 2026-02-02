---
date: 2026-02-01
project: dev-setup
session: ses_3e6014a4dffeWula2ICKAvZdAr
severity: moderate
resolved: true
tags: [opencode, mcp, configuration]
---

## What Happened

OpenCode config validation failed with "Invalid input mcp.mcp-image" error. The MCP server configuration used wrong schema format.

## The Broken Config

```json
"mcp-image": {
  "type": "stdio",           // WRONG
  "command": "npx",          // WRONG
  "args": ["-y", "mcp-image"], // WRONG
  "env": {                   // WRONG field name
    "GEMINI_API_KEY": "...",
    "IMAGE_OUTPUT_DIR": "..."
  }
}
```

## Investigation

1. First tried adding `"type": "stdio"` - still failed
2. Fetched official OpenCode docs at opencode.ai/docs/mcp-servers
3. Discovered correct schema from docs

## Resolution

```json
"mcp-image": {
  "type": "local",                           // local, not stdio
  "command": ["npx", "-y", "mcp-image"],    // array, not separate args
  "environment": {                           // environment, not env
    "GEMINI_API_KEY": "...",
    "IMAGE_OUTPUT_DIR": "..."
  }
}
```

Key differences:
- `type: "local"` not `"stdio"` (OpenCode terminology)
- `command` is a single array including args, no separate `args` field
- `environment` not `env` for env vars

## Lesson

OpenCode MCP schema differs from Claude Desktop's MCP schema. Always check vendor docs - don't assume schema compatibility between different MCP hosts.
