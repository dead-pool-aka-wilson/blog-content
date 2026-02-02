---
title: "How I Automated My Blog's Memory"
date: 2026-02-04
categories: [meta]
tags: [ai-blog-series, opencode, automation, skills, mcp, tooling]
draft: false
summary: "A look at the technical setup that allows my AI assistant to record ideas while I work, without me having to pause or take notes."
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-4.png"
  alt: "Film clapperboard with gear icons"
---

I used to keep a "scratchpad" file that was essentially a graveyard of half-baked ideas. By the time I finished a coding task, I was usually too tired to document the clever bits or the mistakes I made along the way. I realized I was losing my best insights because the friction of manual capture was too high.

This post explains the technical setup I built to solve this. I wanted my terminal to act as a passive observer that notices patterns and records them while I focus on the work itself.

## The Foundation: OpenCode

I chose [OpenCode](https://opencode.ai) as my primary interface. It's an open-source CLI for Claude that runs natively in the terminal. I prefer this over a web UI because it can read my files and run commands directly. 

The two features that made this automation possible are:
1. **Persistent sessions**: I can pick up exactly where I left off.
2. **Tool integration**: The AI can write to my "seeds" directory without me copy-pasting anything.

## Teaching the AI What to Watch For

To make the AI useful, I had to define what "valuable information" looks like. I did this using a "skill"—a markdown file that provides specific instructions to the AI's system prompt.

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

When this skill is active, the AI doesn't just answer my questions. It starts noticing when I say things like "Wait, that's why the cache was failing." It then silently writes those insights to a seed file.

## Removing the Friction

At first, I had to remember to load the skill manually by typing `/content-seed`. This was still too much work. I wanted the harvesting to happen by default.

I updated my configuration to inject this skill into every session:

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

Now, the AI harvests insights without me asking. One of my own comments—"seed idea capture and naming should be automated"—actually triggered the creation of a seed that eventually became part of this article.

## Handling the Backlog

Skills work well during a session, but I also wanted to process sessions I'd already finished. For that, I wrote a small plugin:

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

This ensures that even if I worked on something quickly and forgot to check the seeds, the next time I open the terminal, the system looks back and captures what I missed.

## The Configuration Trap

Setting this up wasn't without hurdles. I spent an hour debugging an MCP (Model Context Protocol) configuration that worked in Claude Desktop but failed in OpenCode.

My first attempt:
```json
"mcp-image": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "mcp-image"],
  "env": { "GEMINI_API_KEY": "..." }
}
```

It failed with a generic "Invalid input" error. After digging through the OpenCode documentation, I found the schema was slightly different:

```json
"mcp-image": {
  "type": "local",                           // not "stdio"
  "command": ["npx", "-y", "mcp-image"],    // single array
  "environment": { "GEMINI_API_KEY": "..." } // not "env"
}
```

The lesson here was simple: don't assume tools that use the same protocol share the same configuration format.

## The Resulting Workflow

Now, the process looks like this:

1. I work on a task.
2. The AI observes and writes seeds to my blog directory.
3. The plugin processes any missed sessions.
4. I later develop those raw seeds into posts like this one.

Most of this is invisible. I focus on solving problems, and the system handles the documentation.

## Is it Worth the Overhead?

Every automation has costs. These skills add more tokens to every prompt, which makes responses slightly slower and more expensive. There are also "false positives" where the AI saves a thought that isn't actually that interesting.

However, for me, the trade-off is clear. I’d rather delete five boring seeds than lose one great idea because I was too busy to write it down. 

## What's Next

The infrastructure is set, and the seeds are flowing. But a seed isn't a post yet. In the next part of this series, I'll show how I turn these raw, unpolished notes into the finished prose you're reading now.

---

*This post was created using insights harvested by the system it describes.*

---

## Series Progress

1. [The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Small Mistakes, Big Lessons](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [How I Automated My Blog's Memory](/posts/2026-02-05-ai-blog-series-4-automation/)
