---
title: "Renaming Projects Is Harder Than You Think"
date: 2026-02-15
author: Raoul
categories: [debugging]
tags: [migration, env-vars, naming, breaking-change]
draft: false
summary: "The gateway failed with 'missing env var.' I had the token—just under the old project name."
cover:
  image: "/covers/cover-2026-02-15-env-var-naming-migration.png"
---

```
MissingEnvVarError: Missing env var "CLAWDBOT_GATEWAY_TOKEN"
referenced at config path: gateway.auth.token
```

I definitely had that token set. I'd used this gateway yesterday. What changed?

Nothing changed on my machine. The *project* changed. It was renamed from `clawdbot` to `openclaw`. The config file still referenced the old environment variable name.

## The Symptom

The project went through multiple renames:
- `clawdbot` → `moltbot` → `openclaw`

Each rename updated the code, the documentation, and the repository. But environment variables live in *user* environments—shell configs, Keychain entries, CI secrets. Those don't auto-update when you push a commit.

My config file:
```json
{
  "gateway": {
    "auth": {
      "token": "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  }
}
```

My environment:
```bash
export OPENCLAW_GATEWAY_TOKEN="actual-token"  # New name
# CLAWDBOT_GATEWAY_TOKEN doesn't exist
```

The config referenced the old name. The error was accurate—that variable really was missing.

## Why Renames Are Deceptive

Renaming a project feels like a find-and-replace operation. It's not. Here's what actually needs updating:

| Artifact | Location | Often Missed |
|----------|----------|--------------|
| Env var prefixes | User shell configs | Yes |
| Config file paths | `~/.oldname/` directories | Yes |
| LaunchAgent labels | `~/Library/LaunchAgents/` | Yes |
| Keychain entries | Account/service names | Yes |
| Symlinks | May point to old directories | Yes |
| CI/CD secrets | GitHub Actions, etc. | Sometimes |
| Documentation | READMEs, wikis | Sometimes |
| Package names | npm, pip, cargo | Usually updated |
| Repository name | GitHub, GitLab | Usually updated |

The visible artifacts (repo, package, docs) get updated because they're in version control. The invisible artifacts (env vars, local configs, system services) stay behind.

## The Fix

1. **Update the config to use new prefix:**
```json
{
  "gateway": {
    "auth": {
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

2. **Rename the Keychain entry:**
```bash
# Delete old entry
security delete-generic-password -a "$USER" -s "clawdbot-gateway-token"

# Add with new name
security add-generic-password -a "$USER" -s "openclaw-gateway-token" -w "actual-token"
```

3. **Update wrapper scripts** that reference the old name.

4. **Add deprecation warnings** to help other users:
```
Warning: Deprecated environment variables detected:
  CLAWDBOT_GATEWAY_TOKEN → Use OPENCLAW_GATEWAY_TOKEN
```

## Prevention: The Rename Checklist

When renaming a project, create a migration guide:

```markdown
## Migration from oldname to newname

### Environment Variables
| Old | New |
|-----|-----|
| OLDNAME_API_KEY | NEWNAME_API_KEY |
| OLDNAME_TOKEN | NEWNAME_TOKEN |

### Config Paths
- `~/.oldname/` → `~/.newname/`
- Config file: `~/.oldname/config.json` → `~/.newname/config.json`

### System Services (macOS)
- LaunchAgent: `com.oldname.daemon` → `ai.newname.daemon`

### Commands
Run: `newname migrate` to auto-update local configs
```

### Backwards Compatibility

For a smoother transition:

```javascript
// Support both old and new env var names
const token = process.env.OPENCLAW_TOKEN || process.env.CLAWDBOT_TOKEN;

if (process.env.CLAWDBOT_TOKEN && !process.env.OPENCLAW_TOKEN) {
  console.warn('CLAWDBOT_TOKEN is deprecated. Use OPENCLAW_TOKEN');
}
```

This gives users time to update while still working.

### Doctor Commands

A `doctor` or `diagnose` command can catch these issues:

```bash
$ openclaw doctor

✓ Config file found
✓ API token valid
⚠ Deprecated env var CLAWDBOT_TOKEN detected
  → Rename to OPENCLAW_TOKEN

Issues found: 1 warning
```

## The Lesson

The code rename was trivial: find and replace. The ecosystem rename was not. Environment variables, Keychain entries, LaunchAgents, and local configs all lived outside version control, still pointing to the old name.

Renaming a project is a breaking change that happens outside your repository. Document it. Provide migration tooling. Support backwards compatibility for at least one version.

The error message was accurate: `CLAWDBOT_GATEWAY_TOKEN` was missing. But the real bug was assuming a project rename would update user environments automatically.
