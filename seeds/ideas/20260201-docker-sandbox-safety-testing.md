---
date: 2026-02-01
session: ses_3e6cce974ffe5iWj9Y1RMrS4pl
tags: [docker, sandbox, security, testing, mac-mini]
stage: seed
content_potential: medium
---

## The Idea

A systematic, phased approach to testing Docker sandbox configurations without risking the host machine - especially important for personal automation bots with access to real data.

## Context

User setting up moltbot on Mac Mini with Docker sandbox. Concerned about:
- Signal-cli (can brick phone identity)
- Volume mounts to personal Obsidian vault
- Email credentials
- Resource limits not actually working

## The Pattern

### Phase 1: Isolated Container Test
```bash
# Use DUMMY mounts, not real data
mkdir -p /tmp/test-sandbox/{downloads,obsidian}
docker run --rm -it \
  --read-only \
  --cpus="2.0" \
  --memory="2g" \
  -v /tmp/test-sandbox/downloads:/mnt/downloads:rw \
  moltbot-sandbox:custom /bin/sh
```

### Phase 2: Resource Limit Validation
```bash
# Stress test that stays inside container limits
docker run --rm -it --cpus="2" --memory="2g" container sh -c "
  stress-ng --cpu 8 --timeout 10s 2>/dev/null || echo 'expected'
"
```

### Phase 3: Credential Testing (Read-Only First)
```bash
# Email: read-only commands only
himalaya list --folder INBOX -s 3  # Safe
# himalaya send ...  # NEVER in testing
```

### Phase 4: High-Risk Components
- Signal-cli: Use burner number or skip entirely
- Backup identity before any testing: `cp -r ~/.signal-cli ~/.signal-cli.backup`

## Raw Exchange

> User: "this machine is mac mini. and sandbox is running in docker. tell me plan to test this config but not blowing up my device"

## Expansion Potential

- Blog: "Safe Testing for Personal AI Bots"
- Checklist template for sandbox verification
- Risk matrix for different integration types
