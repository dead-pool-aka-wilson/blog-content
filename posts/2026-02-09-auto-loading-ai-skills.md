---
title: "Always-On AI Skills: Configuration as Behavior"
date: 2026-02-09
author: Raoul
categories: [technical]
tags: [opencode, configuration, skills, automation]
draft: false
summary: "Stop invoking AI skills manually. Configure them to auto-load, turning one-off commands into persistent background behaviors."
---

Why do I have to remember to invoke `/content-seed` every session?

I built a skill that captures interesting prompts, ideas, and mistakes during AI conversations. It works great—when I remember to load it. Most sessions, I forget. The skill exists, but it's not active, and the seeds don't get captured.

The fix was embarrassingly simple: configuration-based skill injection.

## WHY: Skills as Behaviors, Not Commands

The mental model shift is small but important.

**Command-based skills**: "I want to do X, so I invoke skill X."
- Requires remembering to invoke
- Breaks flow to load the skill
- Works for one-off actions

**Configuration-based skills**: "This agent always has these behaviors."
- Loads automatically at session start
- No interruption to workflow
- Works for persistent behaviors

Content-seed harvesting isn't something I do occasionally—it's something I want happening continuously. The skill should be ambient, not invoked.

## HOW: Auto-Loading via Config

OpenCode (and similar AI assistants) let you configure agent behavior through JSON config files. The key is the `skills` array:

```json
// ~/.config/opencode/oh-my-opencode.json
{
  "agents": {
    "sisyphus": {
      "model": "anthropic/claude-sonnet-4",
      "temperature": 0.3,
      "skills": ["content-seed", "git-workflow"]
    }
  }
}
```

When the `sisyphus` agent starts, it automatically loads `content-seed` and `git-workflow` skills. No manual invocation needed. The behaviors are baked into the agent's identity.

### What This Enables

**Passive monitoring**: Skills like content-seed that observe and capture without explicit commands.

**Consistent enforcement**: Skills like git-workflow that check rules without you remembering to ask.

**Compound agents**: Stack multiple skills to create specialized agent personalities.

```json
{
  "agents": {
    "writer": {
      "skills": ["content-seed", "blog-formatter", "style-guide"]
    },
    "reviewer": {
      "skills": ["code-review", "security-check", "test-coverage"]
    }
  }
}
```

### Skills vs. System Prompts

You could achieve similar results by adding behavior instructions to the system prompt. But skills are better because:

1. **Modular**: Add/remove behaviors without editing prompt text
2. **Reusable**: Same skill works across multiple agents
3. **Versioned**: Skills are files you can track in git
4. **Shareable**: Skills can be published and shared

The config file becomes a declarative definition of agent behavior, not a wall of instructions.

## WHAT: The Implementation Pattern

A skill is typically a markdown file with structured instructions:

```markdown
# Content Seed Skill

## Activation
This skill is always active.

## Behavior
Monitor conversations for:
- Effective prompts (high or low quality)
- Insights and "aha" moments
- Mistakes and debugging lessons

When detected, silently capture to the appropriate seed directory.

## Output Locations
- prompts/ - Notable AI instructions
- ideas/ - Patterns and concepts
- misses/ - Failures and learnings
```

The skill file tells the agent what to do. The config file tells the agent when to load it (always, in this case).

### Debugging Auto-Loaded Skills

When skills load automatically, you can't see the invocation. Debugging tips:

**Check loaded skills at session start**: Most AI assistants will list active skills in their startup message or status.

**Test behavior explicitly**: After starting a session, verify the skill is working by triggering its intended behavior.

**Watch for conflicts**: Multiple skills might have overlapping behaviors. If output looks wrong, check for skill interference.

### When NOT to Auto-Load

Some skills shouldn't be always-on:

- **Heavy skills** that consume lots of context window
- **Specialized skills** for rare workflows (database migration, release management)
- **Experimental skills** you're still developing

Keep the default skill set minimal. Add specialized skills per-session when needed.

## The Takeaway

If you're invoking the same skill every session, you've identified a behavior, not a command. Move it to configuration.

Configuration-as-behavior shifts your relationship with AI assistants. Instead of directing them step-by-step, you define their persistent characteristics. The agent becomes a specialized worker with built-in capabilities, not a blank slate waiting for instructions.

My content-seed harvesting now happens automatically, every session, without me thinking about it. That's the whole point—remove the cognitive overhead of remembering to invoke behaviors I always want.

Skills you invoke manually are tools. Skills that auto-load are teammates.
