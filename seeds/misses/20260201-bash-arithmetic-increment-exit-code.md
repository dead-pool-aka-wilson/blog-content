---
date: 2026-02-01
project: clawdbot-setup
severity: moderate
resolved: true
tags: [bash, gotcha, set-e, arithmetic]
content_potential: medium
---

## What Happened

Test script failed silently after first passing test. The script used `set -e` (exit on error) combined with bash arithmetic increment:

```bash
set -e
TESTS_PASSED=0

log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++))  # THIS CAUSES EXIT!
}
```

## Why It Matters

`((TESTS_PASSED++))` returns exit code based on the POST-increment value:
- When `TESTS_PASSED=0`: `((0++))` evaluates to 0 (falsy) → exit code 1
- `set -e` sees exit code 1 → script exits

This is a common bash gotcha that's hard to debug because:
- No error message
- Happens on the FIRST success, not failure
- Looks like it should work

## Resolution

Use explicit arithmetic assignment instead:

```bash
log_success() {
    echo "[PASS] $1"
    TESTS_PASSED=$((TESTS_PASSED + 1))  # Safe - always returns 0
}
```

## Lesson

Never use `((var++))` with `set -e`. The increment operator's return value is the pre-increment value, which is falsy when starting from 0.
