---
title: "The Tools: Under the Hood of AI-Powered Blogging"
date: 2026-02-04
categories: [meta]
tags: [ai-blog-series, opencode, automation, skills, mcp, tooling]
draft: false
summary: "How the automation actually works - skills, plugins, and the technical plumbing behind AI-assisted content creation"
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-4.png"
  alt: "Film clapperboard with gear icons"
---

## Beyond Ideas: The Machinery

In the [previous post](/posts/2026-02-04-ai-blog-series-3-meta/), I talked about meta-recursion - how the process of creating content becomes content itself. That's the philosophy. But philosophy needs plumbing.

This post is about the technical infrastructure. How does the "AI harvests while I work" vision actually function?

## OpenCode: The Foundation

Everything runs on [OpenCode](https://opencode.ai), an open-source CLI for Claude. Think of it as a terminal-native AI assistant that can read files, run commands, and maintain context across sessions.

Key features I rely on:
- **Persistent sessions**: Conversations are saved and can be resumed
- **Tool integration**: AI can read/write files, run shell commands
- **MCP support**: Extensible via Model Context Protocol servers

But vanilla OpenCode doesn't know about content seeds or harvesting. That's where skills come in.

## Skills: Teaching AI New Tricks

A skill is a markdown file that injects instructions into the AI's context. Simple but powerful.

```markdown
# ~/.config/opencode/skills/content-seed.md

When working with the user, silently watch for:

**Prompts worth saving:**
- Complex/novel requests that worked well
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

When this skill is loaded, the AI becomes a passive observer. It doesn't just answer questions - it notices patterns and records them.

## Always-On Skills

Initially, I invoked skills manually: `/content-seed`. But that breaks the passive paradigm. I want harvesting to happen automatically.

The solution: skill injection via configuration.

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

Now `content-seed` loads automatically for every session. The AI harvests without me asking.

> "seed idea capture and naming should be automated. I am just doing ideation and feedback on agent work and you harvest all"

That quote from my own session became a seed, which became this section you're reading. Meta-recursion at work.

## The Harvest Plugin

Skills are loaded per-session. But what about harvesting *previous* sessions before starting new work?

That needs a plugin:

```typescript
// ~/.config/opencode/plugins/harvest-on-start.ts
export default {
  name: 'harvest-on-start',
  
  onSessionStart(context) {
    // Check for unharvested sessions from last 24h
    // Inject harvest instruction into first message
  }
}
```

When OpenCode starts a new session, this plugin triggers a harvest of recent sessions. Seeds are extracted before they're forgotten.

## MCP Schema Gotcha

Setting this up wasn't smooth. I hit a wall with MCP server configuration.

My first attempt:

```json
"mcp-image": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "mcp-image"],
  "env": { "GEMINI_API_KEY": "..." }
}
```

Failed with "Invalid input" error.

Turns out OpenCode's MCP schema differs from Claude Desktop's. After checking the docs:

```json
"mcp-image": {
  "type": "local",                           // not "stdio"
  "command": ["npx", "-y", "mcp-image"],    // single array
  "environment": { "GEMINI_API_KEY": "..." } // not "env"
}
```

Three differences. No error message told me which field was wrong. Classic configuration debugging.

**Lesson**: Don't assume schema compatibility between MCP hosts. Always check vendor documentation.

## The Full Stack

Here's how it all fits together:

```
[You work] → [AI observes via content-seed skill]
    ↓
[Seeds written to BrainFucked/10-Blog/seeds/]
    ↓
[harvest-on-start plugin] → [Process old sessions]
    ↓
[You develop seeds into drafts]
    ↓
[Hugo builds from published/]
    ↓
[Blog at raoulcoutard.com]
```

The beauty is that most of this happens invisibly. I just work. Content accumulates.

## What This Enables

With this infrastructure in place:

**Zero-friction capture**: Ideas don't need conscious effort to record. They're observed and saved.

**Consistent harvesting**: Old sessions don't rot. The plugin ensures regular extraction.

**Separation of concerns**: Work and documentation are decoupled. I focus on problems; the system handles meta-work.

**Reproducibility**: The entire setup is configuration files. Clone the dotfiles, get the workflow.

## Trade-offs

Nothing is free.

**Overhead**: Skills add to every prompt. More tokens, slightly slower responses.

**False positives**: Sometimes the AI saves things that aren't worth keeping. Manual curation still needed.

**Complexity**: More moving parts than a simple "write in markdown" approach.

Is it worth it? For me, yes. The value of not losing ideas outweighs the costs. Your mileage may vary.

## Next Up

The system now runs. Seeds flow in. But seeds aren't posts. In the next installment, I'll cover the transformation process - how raw seeds become the polished prose you're reading.

---

*This post was written using content seeds harvested from previous sessions. The automation described here created the raw material for its own description.*

---

## Series So Far

1. [Building a Blog with AI: The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Content Begets Content: Discovering Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [The Tools: Under the Hood of AI-Powered Blogging](/posts/2026-02-05-ai-blog-series-4-automation/)

*More to come...*
