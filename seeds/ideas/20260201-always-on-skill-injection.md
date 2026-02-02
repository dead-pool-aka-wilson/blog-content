---
date: 2026-02-01
tags: [opencode, configuration, skills, automation]
stage: seed
content_potential: medium
source: conversation
---

## The Idea

Configure skills to auto-load for specific agents via oh-my-opencode.json.

## Context

Wanted content-seed harvesting to be always active, not requiring manual `/content-seed` invocation each session.

Solution: Add skills array to agent config.

## Implementation

```json
// ~/.config/opencode/oh-my-opencode.json
{
  "agents": {
    "sisyphus": {
      "model": "...",
      "skills": ["content-seed"]  // ← auto-loads
    }
  }
}
```

## Why It Matters

- Skills become persistent behaviors, not one-off invocations
- Enables "always-on" passive agents (harvesters, monitors, etc.)
- Configuration as behavior definition

## Content Angle

- Tutorial: "Making OpenCode skills always-active"
- Pattern: "Passive AI behaviors through skill injection"
