---
title: "Model Not Found: When Config References Don't Match"
date: 2026-02-18
author: Raoul
categories: [debugging]
tags: [opencode, oh-my-opencode, configuration]
draft: false
summary: "OpenCode failed with 'ProviderModelNotFoundError.' The model ID existed—just not spelled the way my config spelled it."
---

```
ProviderModelNotFoundError
providerID: "google"
modelID: "gemini-3-flash"
suggestions: ["gemini-3-flash-preview", "antigravity-gemini-3-flash"]
```

The error was polite enough to suggest alternatives. But why didn't my model ID work when I could see `gemini-3-flash` in the provider's model list?

## The Symptom

My `oh-my-opencode.json` configured custom agents:

```json
{
  "agents": {
    "multimodal-looker": {
      "model": "google/gemini-3-flash"
    },
    "visual-engineering": {
      "model": "google/gemini-3-flash"
    }
  }
}
```

OpenCode refused to start. Every agent referencing `gemini-3-flash` failed validation.

## The Investigation

The error's `suggestions` field was the key. It showed valid model IDs:
- `gemini-3-flash-preview`
- `antigravity-gemini-3-flash`

These weren't the same as `gemini-3-flash`. Close, but not identical.

Checking the provider config in `opencode.json`:

```json
{
  "providers": {
    "google": {
      "models": [
        { "id": "antigravity-gemini-3-flash", "..." },
        { "id": "antigravity-gemini-3-pro", "..." }
      ]
    }
  }
}
```

The provider defined models with an `antigravity-` prefix. My config omitted the prefix.

## The Fix

Updated all model references to use the exact IDs from the provider config:

```json
{
  "agents": {
    "multimodal-looker": {
      "model": "google/antigravity-gemini-3-flash"
    },
    "visual-engineering": {
      "model": "google/antigravity-gemini-3-flash"
    }
  }
}
```

Four occurrences across different agent configs. All needed the full ID.

## Why This Matters

Model IDs in configuration must exactly match the provider definitions. There's no fuzzy matching:
- `gemini-3-flash` ≠ `antigravity-gemini-3-flash`
- Partial matches don't work
- Case sensitivity may apply

The config system treats model IDs as opaque strings. If the string doesn't exactly match a defined model, it fails.

## The Pattern

This happens whenever two config files reference each other:

```
opencode.json         oh-my-opencode.json
─────────────         ───────────────────
providers:            agents:
  google:               visual-engineering:
    models:               model: "google/???"
      - id: "xxx"                      ↑
                         Must match exactly
```

The agent config references models by ID. Those IDs must exist in the provider config. There's no validation at write time—only at load time.

## Where Model IDs Live

Understanding the config hierarchy helps prevent this error:

```text
~/.config/opencode/
├── opencode.json           # Defines providers and their models
│   └── providers.google.models[].id = "antigravity-gemini-3-flash"
│
└── oh-my-opencode.json     # References models by ID
    └── agents.visual.model = "google/???"
                                    ↑
                          Must match opencode.json exactly
```

The model ID is defined in one place (`opencode.json`) and referenced in another (`oh-my-opencode.json`). The reference must use the exact string from the definition.

### Common ID Drift Scenarios

| Scenario | What Goes Wrong |
|----------|-----------------|
| Copied from docs | Docs show base name, your provider uses prefixed name |
| Copied from another config | Other config uses different provider setup |
| Model renamed | Provider updated, your config didn't |
| Typo | `gemini-3-falsh` vs `gemini-3-flash` |

## Prevention

### 1. Use the error's suggestions
The `suggestions` field shows valid alternatives. Copy-paste from there:

```text
suggestions: ["gemini-3-flash-preview", "antigravity-gemini-3-flash"]
```

### 2. List available models before configuring
```bash
# If your tool supports it:
opencode models list google

# Or grep the config directly:
grep -A2 '"id":' ~/.config/opencode/opencode.json
```

### 3. Cross-reference config files
When editing agent configs, have the provider config open. Verify IDs match exactly.

### 4. Use variables/references if supported
Some config systems allow referencing:
```json
{
  "defaults": {
    "fastModel": "google/antigravity-gemini-3-flash"
  },
  "agents": {
    "visual": { "model": "${defaults.fastModel}" }
  }
}
```

This centralizes the model ID, reducing copy-paste errors.

### 5. Add a validation script
```bash
#!/bin/bash
# validate-models.sh - Check that agent configs reference valid models

DEFINED=$(grep -oP '"id":\s*"\K[^"]+' ~/.config/opencode/opencode.json)
REFERENCED=$(grep -oP '"model":\s*"[^/]+/\K[^"]+' ~/.config/opencode/oh-my-opencode.json)

for model in $REFERENCED; do
  if ! echo "$DEFINED" | grep -q "^$model$"; then
    echo "INVALID: $model"
  fi
done
```

Run this before restarting OpenCode to catch mismatches early.

## The Takeaway

The error was accurate: `gemini-3-flash` wasn't found because it doesn't exist. The model that exists is `antigravity-gemini-3-flash`. Same model, different ID.

When config references fail:
1. Check the exact string being referenced
2. Check the exact string that exists
3. Don't assume partial matches work

Configuration is string matching. Close doesn't count.
