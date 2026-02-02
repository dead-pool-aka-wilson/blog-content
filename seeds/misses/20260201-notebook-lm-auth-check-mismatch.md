---
date: 2026-02-01
project: moltbot/notebook-lm skill
session: ses_3e5ce2650ffezITFBkVK17PzIN
severity: moderate
resolved: true
tags: [playwright, auth, integration-bug, mismatch]
---

## What Happened

NotebookLM upload skill failed with "Auth required" even after user completed browser auth flow.

## Investigation

Shell script (`notebook.sh`) checked for auth like this:
```bash
AUTH_FILE="$AUTH_DIR/notebook-lm-auth.json"
if [[ ! -f "$AUTH_FILE" ]]; then
    echo "Auth required..."
fi
```

But TypeScript upload script (`upload.ts`) actually used Chrome profile:
```typescript
const AUTH_DIR = join(homedir(), ".config", "moltbot", "notebook-lm-chrome");
// ... launches browser with userDataDir
```

The auth flow saved a **Chrome profile directory**, not a JSON file. The shell script checked for the wrong artifact.

## Resolution

Fixed shell script to check for Chrome profile instead:
```bash
CHROME_PROFILE="$AUTH_DIR/notebook-lm-chrome/Default"
if [[ ! -d "$CHROME_PROFILE" ]]; then
    echo "Auth required..."
fi
```

## Lesson

**When integrating scripts in different languages, verify the actual artifacts each component creates.**

Common pattern: Shell wrapper → TypeScript implementation. The wrapper often makes assumptions about what the implementation does. Always trace the actual file/directory creation path.

## Debugging Tip

```bash
# Before: "Why doesn't auth work?"
ls -la ~/.config/moltbot/notebook-lm*
# Reveals: notebook-lm-chrome/ exists, notebook-lm-auth.json does not
```
