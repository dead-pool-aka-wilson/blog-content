---
title: "The Auth File That Didn't Exist"
date: 2026-02-16
author: Raoul
categories: [debugging]
tags: [playwright, auth, integration-bug, mismatch]
draft: false
summary: "I completed the browser auth flow. The script still said 'Auth required.' The shell wrapper and TypeScript implementation disagreed about what 'authenticated' meant."
cover:
  image: "/covers/cover-2026-02-16-playwright-auth-mismatch.png"
---

"Auth required. Please complete the browser authentication flow."

I had just completed the browser authentication flow. The Google login succeeded. NotebookLM opened. The browser saved its state. But the upload script still demanded authentication.

## The Symptom

The NotebookLM automation had two parts:
1. A shell wrapper (`notebook.sh`) that checked auth status
2. A TypeScript script (`upload.ts`) that did the actual browser automation

The auth flow completed successfully. But re-running the command produced the same "Auth required" message every time.

## The Investigation

The shell script checked for auth like this:

```bash
AUTH_DIR="$HOME/.config/moltbot"
AUTH_FILE="$AUTH_DIR/notebook-lm-auth.json"

if [[ ! -f "$AUTH_FILE" ]]; then
    echo "Auth required. Please complete browser authentication."
    exit 1
fi
```

Clear enough: look for a JSON file, fail if missing.

The TypeScript implementation:

```typescript
const AUTH_DIR = join(homedir(), ".config", "moltbot", "notebook-lm-chrome");

const browser = await chromium.launchPersistentContext(AUTH_DIR, {
  headless: false,
  // ... options
});
```

The TypeScript used a Chrome profile directory for persistence, not a JSON file. When you authenticate in the browser, Playwright saves cookies, localStorage, and session data to that directory. No JSON file gets created.

The shell wrapper checked for `notebook-lm-auth.json`.
The TypeScript created `notebook-lm-chrome/`.

Different artifacts. Different paths. The wrapper would never see the auth that the implementation created.

## The Fix

Updated the shell script to check for the Chrome profile:

```bash
CHROME_PROFILE="$AUTH_DIR/notebook-lm-chrome/Default"

if [[ ! -d "$CHROME_PROFILE" ]]; then
    echo "Auth required. Please complete browser authentication."
    exit 1
fi
```

The `Default/` subdirectory is created by Chromium when the profile is first used. If it exists, authentication has been completed.

## Why This Happened

The shell wrapper was written first, with assumptions about how auth would work. The TypeScript implementation came later and used Playwright's persistent context feature—which stores auth differently than the wrapper expected.

Neither component was wrong individually. They just disagreed about the contract between them.

## The Broader Lesson

When integrating scripts across languages, verify the actual artifacts:

```bash
# Before: "Why doesn't auth work?"
# After: "What files actually exist?"

ls -la ~/.config/moltbot/notebook-lm*
# Reveals the truth:
# notebook-lm-chrome/     <- Directory exists (TypeScript creates this)
# notebook-lm-auth.json   <- Missing (shell expects this)
```

Common integration mismatches:

| Wrapper Expects | Implementation Creates |
|-----------------|----------------------|
| JSON config file | Directory structure |
| Single auth token | Cookie database |
| Environment variable | File on disk |
| Specific filename | Timestamped filename |

The fix is always the same: trace what the implementation actually produces, then update the wrapper to check for that.

## Prevention Pattern

When writing wrappers around implementations:

1. **Don't assume** - Run the implementation and see what it creates
2. **Check actual paths** - `ls -la` before writing checks
3. **Document the contract** - "Auth creates `~/.config/app/chrome-profile/`"
4. **Use implementation's constants** - Import paths from the implementation if possible

```bash
# Better: derive from implementation
CHROME_PROFILE=$(node -e "console.log(require('./upload').AUTH_DIR)")
if [[ ! -d "$CHROME_PROFILE/Default" ]]; then
    echo "Auth required"
fi
```

The bug wasn't in the auth flow. The auth flow worked perfectly. The bug was in the mismatch between what the shell expected and what TypeScript provided.

When "it should work" doesn't work, check the assumptions about what "working" produces.
