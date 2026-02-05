---
title: "How I Automated My Blog's Memory"
date: 2026-02-05
author: Raoul
categories: [meta]
tags: [ai-blog-series, opencode, automation, skills, mcp, tooling]
draft: false
summary: "Why I automated knowledge capture, and the technical setup that lets AI record ideas while I work."
lang: en
cover:
  image: "/covers/cover-2026-02-05-ai-blog-series-4-automation.png"
series: ["Building a Blog with AI"]
---

## Why: Friction is the Graveyard of Ideas

I used to keep a "scratchpad" file that was essentially a graveyard of half-baked ideas. By the time I finished a coding task, I was usually too tired to document the clever bits or the mistakes I made along the way. I realized I was losing my best insights because the friction of manual capture was too high.

Friction kills capture. If you have to stop what you're doing, open a new file, and context-switch to "note-taking mode," you probably won't do it. To preserve the evolution of a project, the documentation must be a byproduct of the work, not a separate task. I wanted my environment to act as a passive observer—a system that notices patterns and records them while I focus on solving the problem at hand.

## How: Teaching the Assistant to Watch

The bridge between "working" and "recording" is an AI assistant that understands the value of what's happening in the terminal. I use [OpenCode](https://opencode.ai) as my primary interface because it runs natively in my dev environment, allowing it to see my files and execution history.

To make this passive observation work, I had to teach the AI what to watch for. I did this through "skills"—instructional modules that define what constitutes a "seed" worth saving. By injecting these instructions into every session, the AI stops being just a code generator and starts acting as a digital archivist.

## What: The Technical Implementation

### 1. The Harvesting Skill
I defined a skill that tells the AI exactly what to capture: prompts that worked, "aha!" moments, and even debugging failures.

```markdown
# ~/.config/opencode/skills/content-seed.md

When working with the user, silently watch for:

**Prompts worth saving:**
- Complex requests that worked well
- Meta-prompts about AI usage

**Ideas worth capturing:**
- "I just realized..." moments
- Patterns discovered during work
- Design decisions with interesting rationale

**Misses to record:**
- Bugs and their fixes
- Wrong approaches tried
- Debugging journeys

Write seeds to: ~/Dev/BrainFucked/10-Blog/seeds/
```

### 2. Always-On Configuration
To remove the friction of manual activation, I updated my global configuration to inject this skill into every "Sisyphus" (executor) session automatically.

```json
// ~/.config/opencode/oh-my-opencode.json
{
  "agents": {
    "sisyphus": {
      "skills": ["content-seed"]
    }
  }
}
```

### 3. Backlog Processing with Plugins
For sessions where I worked too fast or used a different agent, I wrote a `harvest-on-start` plugin. It scans the session history from the last 24 hours and prompts the AI to "harvest" any missed insights at the start of a new session.

```typescript
// ~/.config/opencode/plugins/harvest-on-start.ts
export default {
  name: 'harvest-on-start',
  onSessionStart(context) {
    // Logic to check for unharvested sessions
    // and inject harvest instructions
  }
}
```

### 4. The MCP Configuration Trap
Integrating these tools requires careful configuration of the Model Context Protocol (MCP). I lost an hour debugging a setup that worked in Claude Desktop but failed in OpenCode because of subtle schema differences.

**The Failure:**
```json
"mcp-image": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "mcp-image"],
  "env": { "GEMINI_API_KEY": "..." }
}
```

**The Fix:**
```json
"mcp-image": {
  "type": "local",                           // OpenCode uses 'local'
  "command": ["npx", "-y", "mcp-image"],    // Array for the full command
  "environment": { "GEMINI_API_KEY": "..." } // 'environment', not 'env'
}
```

## The Resulting Workflow

Now, my workflow is invisible:
1. **Work**: I solve problems in the terminal via OpenCode.
2. **Observe**: The AI notices a relevant pattern or fix.
3. **Capture**: It silently writes a "seed" file to my blog content directory.
4. **Develop**: I later review these raw seeds and expand them into posts.

I’d rather delete five boring seeds than lose one great idea because I was too busy to write it down. By automating the memory of my blog, I can focus on the work, knowing the story is being told in the background.

---

*This post was created using insights harvested by the system it describes.*

---

## Series Progress

1. [The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Small Mistakes, Big Lessons](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [How I Automated My Blog's Memory](/posts/2026-02-05-ai-blog-series-4-automation/)
