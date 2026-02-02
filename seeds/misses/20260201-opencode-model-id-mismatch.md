---
date: 2026-02-01
project: dev-setup
session: ses_3e6014a4dffeWula2ICKAvZdAr
severity: minor
resolved: true
tags: [opencode, oh-my-opencode, configuration]
---

## What Happened

ProviderModelNotFoundError when starting OpenCode. The `oh-my-opencode.json` config referenced model IDs that don't exist in the provider definitions.

## The Error

```
ProviderModelNotFoundError
providerID: "google"
modelID: "gemini-3-flash"
suggestions: ["gemini-3-flash-preview", "antigravity-gemini-3-flash"]
```

## Investigation

1. Error showed suggestions for valid model names
2. Searched config files for the invalid model ID
3. Found references in `~/.config/opencode/oh-my-opencode.json`:
   - Line 19: `multimodal-looker` agent
   - Line 39: `visual-engineering` category
   - Line 46: `artistry` category
   - Line 60: `writing` category

## Resolution

Changed all occurrences to use valid model IDs:

```json
// Before (invalid)
"model": "google/gemini-3-flash"
"model": "google/gemini-3-pro"

// After (valid - using antigravity provider)
"model": "google/antigravity-gemini-3-flash"
"model": "google/antigravity-gemini-3-pro"
```

## Lesson

Model IDs in `oh-my-opencode.json` must exactly match the model IDs defined in `opencode.json` provider config. The error's `suggestions` field shows valid alternatives - use those exact strings.
