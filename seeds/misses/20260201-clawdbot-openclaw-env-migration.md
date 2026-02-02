---
date: 2026-02-01
project: openclaw
session: ses_3e6cce974ffe5iWj9Y1RMrS4pl
severity: minor
resolved: true
tags: [migration, env-vars, naming, breaking-change]
---

## What Happened

OpenClaw gateway failed to start with:
```
MissingEnvVarError: Missing env var "CLAWDBOT_GATEWAY_TOKEN" 
referenced at config path: gateway.auth.token
```

User had the token set, but under wrong name.

## Investigation

Project was renamed: `clawdbot` → `moltbot` → `openclaw`

Config file (`~/.clawdbot/moltbot.json`) still referenced old env var:
```json
"gateway": {
  "auth": {
    "token": "${CLAWDBOT_GATEWAY_TOKEN}"
  }
}
```

But the new convention uses `OPENCLAW_*` prefix.

## Resolution

1. Updated config to use `${OPENCLAW_GATEWAY_TOKEN}`
2. Renamed env var in Keychain and wrapper script
3. Doctor command now warns about deprecated vars:

```
- Deprecated legacy environment variables detected (ignored).
- Use OPENCLAW_* equivalents instead:
  CLAWDBOT_GATEWAY_TOKEN -> OPENCLAW_GATEWAY_TOKEN
```

## Lesson

**Renaming projects is deceptively hard.** Things that need updating:

| Artifact | Often Missed |
|----------|--------------|
| Env var prefixes | `PROJECT_*` → `NEWNAME_*` |
| Config paths | `~/.oldname/` → `~/.newname/` |
| LaunchAgent labels | `com.oldname.` → `ai.newname.` |
| Keychain entries | Account/service names |
| Symlinks | May point to old locations |

OpenClaw's doctor command handles some of this via auto-migration, but env vars in user scripts/configs require manual updates.

## Prevention

When renaming a project:
1. Provide migration command or documentation
2. Keep backwards-compat aliases for 1-2 versions
3. Log warnings for deprecated names (don't just fail)
