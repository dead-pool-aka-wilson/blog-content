---
date: 2026-02-01
tags: [security, macos, keychain, api-keys, secrets-management]
stage: seed
content_potential: high
source: conversation
---

## The Idea

Use a shell wrapper script that fetches API keys from macOS Keychain only when the command actually runs - never exposing secrets in shell environment globally.

## Context

Setting up OpenClaw/moltbot automation. User correctly pushed back on storing API keys in ~/.zshrc because:
- Keys visible to ALL processes
- Persists in shell history
- Often committed to dotfiles repos

## Raw Quote

> "putting api in shell config seems not good"

## The Pattern

```bash
#!/bin/bash
# ~/bin/openclaw - wrapper that fetches from Keychain on-demand

fetch_key() {
    local keychain_name="$1"
    security find-generic-password -a "$USER" -s "$keychain_name" -w 2>/dev/null || {
        echo "Error: $keychain_name not found in Keychain" >&2
        return 1
    }
}

export GEMINI_API_KEY=$(fetch_key "gemini-api-key") || exit 1
export OPENCLAW_GATEWAY_TOKEN=$(fetch_key "openclaw-gateway-token") || exit 1

exec /path/to/actual/command "$@"
```

## Why This Is Better

| Approach | Key Exposure |
|----------|--------------|
| ~/.zshrc export | Every shell, every process |
| Wrapper script | Only during command execution |

## Expansion Potential

- Blog post: "The Right Way to Handle API Keys on macOS"
- Could apply to any CLI tool needing secrets
- Universal pattern for security-conscious developers
