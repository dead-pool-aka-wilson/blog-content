---
title: "The Bash Increment That Killed My Script"
date: 2026-02-14
author: Raoul
categories: [debugging]
tags: [bash, gotcha, set-e, arithmetic]
draft: false
summary: "My test script died silently after the first passing test. The culprit: ((var++)) combined with set -e."
---

The script worked perfectly—for exactly one test.

I wrote a test runner with `set -e` to catch failures. Each passing test incremented a counter. The first test passed, logged success, and then... nothing. The script exited silently. No error message. No indication of what went wrong.

## The Symptom

```bash
#!/bin/bash
set -e

TESTS_PASSED=0

log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++))  # Script dies here
}

# Tests
log_success "Test 1"  # Prints "[PASS] Test 1", then exits
log_success "Test 2"  # Never reached
log_success "Test 3"  # Never reached

echo "Passed: $TESTS_PASSED"  # Never reached
```

Run this script. You'll see `[PASS] Test 1` and then your prompt returns. No "Test 2", no final count. The script exits after the first increment.

## The Investigation

Adding `set -x` for debugging showed the script exiting right after `((TESTS_PASSED++))`. But why? The increment should be harmless.

The insight came from understanding how bash arithmetic evaluation works:

```bash
TESTS_PASSED=0
((TESTS_PASSED++))
echo $?  # Prints "1" - failure!
```

The `(( ))` construct returns an exit code based on the expression's value:
- Non-zero result → exit code 0 (success)
- Zero result → exit code 1 (failure)

`((TESTS_PASSED++))` is a post-increment. It returns the value *before* incrementing. When `TESTS_PASSED` is 0, the expression evaluates to 0, which bash treats as false, which means exit code 1.

With `set -e`, any command returning non-zero exits the script. The increment "succeeds" (the variable updates), but the return value is falsy, so `set -e` kills the script.

## The Fix

Use explicit arithmetic assignment instead of the increment operator:

```bash
log_success() {
    echo "[PASS] $1"
    TESTS_PASSED=$((TESTS_PASSED + 1))  # Always returns 0
}
```

Or suppress the exit code:

```bash
log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++)) || true  # Force success
}
```

Or use the pre-increment (only works if you never start from zero):

```bash
TESTS_PASSED=-1  # Hacky, don't do this
((++TESTS_PASSED))  # Pre-increment returns new value
```

The cleanest fix is the first: `VAR=$((VAR + 1))` has no exit code gotchas.

## Why This Is Evil

This bug is hard to catch because:

1. **No error message** - The script just stops
2. **It fails on success** - The first *passing* test kills it
3. **The code looks correct** - `((var++))` is idiomatic bash
4. **It's timing-dependent** - Only fails when starting from 0

If your counter starts at 1, or if you're decrementing, or if you've already incremented once, you'll never see this bug. It only triggers on the first increment from zero.

## The Broader Lesson

`set -e` is useful but dangerous. It turns minor bash quirks into script-terminating landmines. Other `set -e` gotchas:

```bash
# These all have surprising exit codes:
local var=$(false)      # Local succeeds even if command fails
var=$(false) || true    # The || true is never reached
grep "pattern" file     # Exit 1 if pattern not found (not error)
diff file1 file2        # Exit 1 if files differ (not error)
```

The safest approach: use `set -e` for critical scripts, but explicitly handle commands with tricky exit codes.

```bash
set -e

# Explicitly handle potential "failures"
count=$(grep -c "pattern" file) || count=0
TESTS_PASSED=$((TESTS_PASSED + 1))  # Safe increment
```

## The Takeaway

I spent 30 minutes debugging a test framework that was being killed by its own success counter. The fix was changing `((var++))` to `var=$((var + 1))`.

Bash is full of these sharp edges. The increment operator's return value behavior is documented, but it's not intuitive, and it interacts badly with `set -e`.

When your script dies silently after doing exactly what it should, suspect an exit code gotcha. And never trust `((var++))` in a `set -e` script.
