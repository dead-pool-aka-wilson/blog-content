---
title: "Safe Testing for AI Bot Sandboxes: A Phased Approach"
date: 2026-02-10
author: Raoul
categories: [technical]
tags: [docker, sandbox, security, testing, automation]
draft: false
summary: "When your AI bot can send texts and access real files, how do you test the sandbox without blowing up your actual data? A four-phase approach."
---

The bot had access to my phone's Signal identity and my Obsidian vault. How do I test that the sandbox actually contains it?

I was setting up moltbot—a personal automation bot—inside a Docker container on my Mac Mini. The bot needs to send notifications via Signal, read and write to my notes vault, and access email. Each capability is also a risk: wrong configuration could brick my Signal identity, corrupt notes, or send emails I didn't intend.

Docker isolation is supposed to prevent this. But "supposed to" isn't good enough when real data is on the line.

## WHY: Trust But Verify

Docker resource limits and volume mounts look right in the config file. But until you test them, you don't know:

- Do CPU limits actually constrain the container?
- Can the container write outside its mounted volumes?
- Will read-only flags actually prevent writes?

The config says what should happen. Testing confirms what does happen.

## HOW: Four Phases of Sandbox Testing

### Phase 1: Isolated Container with Dummy Data

Never test with real data first. Create fake mount points:

```bash
# Create test directories with dummy content
mkdir -p /tmp/test-sandbox/{downloads,obsidian}
echo "test note" > /tmp/test-sandbox/obsidian/test.md

# Run container with dummy mounts, not real paths
docker run --rm -it \
  --read-only \
  --cpus="2.0" \
  --memory="2g" \
  -v /tmp/test-sandbox/downloads:/mnt/downloads:rw \
  -v /tmp/test-sandbox/obsidian:/mnt/obsidian:ro \
  moltbot-sandbox:latest /bin/sh
```

Inside the container, try to:
- Write to the read-only mount (should fail)
- Write outside mounted volumes (should fail)
- Confirm you can read from mounted paths

If any of these don't behave as expected, fix the config before touching real data.

### Phase 2: Resource Limit Validation

Config says `--cpus="2.0"` and `--memory="2g"`. Does it actually limit?

```bash
docker run --rm -it \
  --cpus="2" \
  --memory="2g" \
  moltbot-sandbox:latest sh -c "
    # Try to use 8 CPUs - should be throttled to 2
    stress-ng --cpu 8 --timeout 10s 2>/dev/null && echo 'Completed' || echo 'Expected limitation'
    
    # Check visible CPU count
    nproc
    
    # Check memory limit
    cat /sys/fs/cgroup/memory/memory.limit_in_bytes 2>/dev/null || \
    cat /sys/fs/cgroup/memory.max 2>/dev/null
  "
```

Watch `docker stats` in another terminal while the stress test runs. You should see CPU usage capped at your limit.

### Phase 3: Credential Testing (Read-Only Operations)

Before testing send operations, verify read operations work with real credentials:

```bash
# Email: read-only operations only
himalaya list --folder INBOX --page-size 3  # Safe: just lists
himalaya read 1 --headers  # Safe: reads one message

# NEVER during testing:
# himalaya send ...
# himalaya forward ...
```

Read operations prove the credentials work without risking unintended actions. Only proceed to write operations after confirming the sandbox contains them properly.

### Phase 4: High-Risk Components in Isolation

Some components are too dangerous to test casually. Signal-cli, for example, manages your phone's identity. Wrong configuration can brick it.

**Before testing Signal:**
```bash
# Backup identity completely
cp -r ~/.signal-cli ~/.signal-cli.backup.$(date +%Y%m%d)
```

**During testing:**
- Use a burner number if possible
- Start with receive-only operations
- Test send to yourself first
- Never test group operations until everything else works

**If something breaks:**
```bash
# Restore from backup
rm -rf ~/.signal-cli
cp -r ~/.signal-cli.backup.$(date +%Y%m%d) ~/.signal-cli
```

## WHAT: The Risk Matrix

Categorize each integration by risk before testing:

| Integration | Read Risk | Write Risk | Recovery |
|-------------|-----------|------------|----------|
| File mounts | Low | Medium | Restore from backup |
| Email | Low | High | Visible but awkward |
| Signal | Medium | Critical | Backup required |
| Calendar | Low | Medium | Events can be deleted |
| Database | Medium | Critical | Backup required |

**High write-risk** integrations get extra phases:
1. Test with mocks/stubs
2. Test read operations only
3. Test write to self/test destination
4. Test actual operations with backup ready

### Automation Testing Checklist

Before considering any sandbox configuration "tested":

- [ ] Container cannot write outside defined mounts
- [ ] Read-only mounts actually prevent writes
- [ ] Resource limits are enforced (not just configured)
- [ ] Network isolation works if configured
- [ ] Credentials work for read operations
- [ ] Backups exist for high-risk integrations
- [ ] Send/write operations tested with safe destinations first

## The Takeaway

The question wasn't "will Docker sandbox work?" It was "how do I prove it works without destroying real data in the process?"

Phased testing answers that. Start with fake data. Verify constraints actually constrain. Test read before write. Backup everything critical before high-risk operations.

My moltbot now runs in a sandbox I've actually verified, not one I hope is configured correctly. The hour spent on phased testing beats the day spent recovering from a "it should have been fine" moment.

When your bot has access to real data, hope isn't a security strategy. Test the cage before you trust it.
