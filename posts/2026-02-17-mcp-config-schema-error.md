---
title: "MCP Config Schemas: Not All Hosts Are Equal"
date: 2026-02-17
author: Raoul
categories: [debugging]
tags: [opencode, mcp, configuration]
draft: false
summary: "My MCP server config worked in Claude Desktop. It failed validation in OpenCode. Same protocol, different schemas."
cover:
  image: "/covers/cover-2026-02-17-mcp-config-schema-error.png"
---

```
Invalid input mcp.mcp-image
```

The config looked right. It matched the MCP examples I'd seen. It worked in Claude Desktop. But OpenCode rejected it on startup.

## The Symptom

I copied an MCP server configuration that worked elsewhere:

```json
{
  "mcp": {
    "mcp-image": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-image"],
      "env": {
        "GEMINI_API_KEY": "...",
        "IMAGE_OUTPUT_DIR": "..."
      }
    }
  }
}
```

OpenCode's validation failed immediately. No helpful error message—just "Invalid input."

## The Investigation

First attempt: maybe I needed `"type": "stdio"` explicitly? Added it. Still failed.

The breakthrough came from reading OpenCode's actual documentation instead of assuming MCP configs were universal. The schema was different:

**Claude Desktop / Cline format:**
```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "mcp-image"],
  "env": { ... }
}
```

**OpenCode format:**
```json
{
  "type": "local",
  "command": ["npx", "-y", "mcp-image"],
  "environment": { ... }
}
```

Three differences:
1. `type: "local"` instead of `type: "stdio"`
2. `command` is an array including all args, no separate `args` field
3. `environment` instead of `env`

## The Fix

```json
{
  "mcp": {
    "mcp-image": {
      "type": "local",
      "command": ["npx", "-y", "mcp-image"],
      "environment": {
        "GEMINI_API_KEY": "...",
        "IMAGE_OUTPUT_DIR": "..."
      }
    }
  }
}
```

Validation passed. Server started.

## Why Schemas Differ

MCP (Model Context Protocol) defines the wire protocol—how the host and server communicate. It doesn't mandate how hosts should configure servers.

Each host implements its own configuration layer:
- **Claude Desktop**: JSON with `command` + `args` separation
- **OpenCode**: JSON with `command` as array
- **Cline**: Similar to Claude Desktop
- **Continue**: YAML-based config

The MCP spec covers message formats and capabilities. Configuration is a host implementation detail.

## The Broader Lesson

Don't assume config portability across MCP hosts. When moving servers between hosts:

1. **Find the host's docs** - Not generic MCP docs
2. **Check schema differences** - Field names, value types, structure
3. **Test in isolation** - Validate config before assuming it works

Common differences across hosts:

| Aspect | May Vary |
|--------|----------|
| Type names | `stdio` vs `local` vs `process` |
| Command format | String vs array vs object |
| Env var field | `env` vs `environment` vs `envVars` |
| Args handling | Separate field vs in command array |
| Working dir | Different field names |

## Prevention Pattern

Keep host-specific configs in separate files:

```text
~/.config/
├── claude-desktop/
│   └── mcp-servers.json     # Claude format
├── opencode/
│   └── opencode.json        # OpenCode format
└── mcp-servers/
    └── mcp-image/
        └── README.md        # Server-specific notes
```

When adding a new server, don't copy from other hosts. Start from the host's documentation or example configs.

### Minimal Configs That Load

When debugging, start with the simplest possible config:

**OpenCode minimal:**
```json
{
  "mcp": {
    "echo-test": {
      "type": "local",
      "command": ["echo", "hello"]
    }
  }
}
```

**Claude Desktop minimal:**
```json
{
  "mcpServers": {
    "echo-test": {
      "command": "echo",
      "args": ["hello"]
    }
  }
}
```

If the minimal config loads, add fields one at a time until you find the problematic one.

### Validation Checklist

Before starting the host:

```bash
# 1. Check JSON syntax
python3 -m json.tool < opencode.json > /dev/null && echo "JSON OK"

# 2. Check required fields exist
jq '.mcp | to_entries[] | select(.value.type == null) | .key' opencode.json
# Should return nothing if all servers have "type"

# 3. Check command is array (OpenCode)
jq '.mcp | to_entries[] | select(.value.command | type != "array") | .key' opencode.json
# Should return nothing
```

## The Takeaway

MCP is a protocol, not a configuration standard. Each host that speaks MCP can configure servers however it wants. The wire format is compatible; the config files are not.

The error "Invalid input" was correct—my input was invalid for OpenCode's schema. It just happened to be valid for a different host's schema.

When "it works elsewhere" doesn't work here, you're looking at a schema mismatch, not a broken server.

When config works in one place but fails in another, check if you're using the right schema for the right host. Same protocol doesn't mean same config.
