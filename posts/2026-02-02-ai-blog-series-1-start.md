---
title: "Building a Blog with AI: The Content Seed System"
date: 2026-02-02
author: Raoul
categories: [meta]
tags: [ai-blog-series, opencode, blog, content-creation, hugo, cloudflare]
draft: false
summary: "Why I named this blog after a cinematographer, chose Hugo and Cloudflare, and built a system to harvest AI conversations into posts."
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-1.png"
  alt: "Film clapperboard with play button"
---

How much of your best thinking currently rots in a chat history you'll never open again?

Every day, I spend hours talking to AI. We debug race conditions, debate architecture, and solve edge cases. Until recently, these insights vanished the moment I closed the session. I was leaking intellectual capital. 

I needed a system to capture raw work as it happens, not a platform for polished performance.

## WHY: The Philosophy

### Raoul Coutard and "Uncut"
This blog is named after Raoul Coutard (1924-2016), the French cinematographer who revolutionized New Wave cinema. A former war photojournalist with no formal film training, he broke every convention of his time. He used handheld cameras, natural light, and worked five times faster than traditional crews by keeping it simple.

Coutard’s documentary realism is the blueprint for this blog. AI-assisted content creation breaks traditional blogging rules. I don't "write" posts; I capture raw sessions. Records don't edit. Like a Coutard film, the goal is to show the work as it is: handheld, available light, and fast.

### Choosing the Tools
I chose my stack based on the same philosophy of simplicity and speed:
- **WHY Hugo**: It’s a static site generator. No runtime, no database, just files. It’s markdown-native and builds in milliseconds. I get power without the overhead of a framework.
- **WHY Cloudflare Pages**: It provides global edge deployment with a generous free tier. It integrates with Hugo via the Wrangler CLI and triggers builds automatically on git push. No vendor lock-in; the output is just standard HTML.

## HOW: The System

### Separated Repositories
I maintain two distinct repositories to keep infrastructure and creative output separate:
1. **personal-blog-hosting**: Infrastructure (Hugo config, theme, deployment).
2. **blog-content** (submodule): All posts, seeds, and skills.

**WHY split?** Content lives in an Obsidian vault (**BrainFucked**) for daily knowledge management. Infrastructure changes shouldn't pollute my content history. This separation allows me to reuse the same content across different projects without friction.

### The Content Seed System
Instead of writing from scratch, I harvest "seeds" from AI conversations:
- **Prompts**: Effective instructions that delivered results.
- **Ideas**: Raw insights that happen mid-coding.
- **Misses**: Failures, bugs, and lessons learned.

By directing the AI to identify these seeds during a session, I stop the evaporation of thought.

## WHAT: The Implementation

The result is a structured workspace where the blog literally builds itself from my daily work:

```text
blog-content/
├── seeds/
│   ├── prompts/     # Notable AI requests
│   ├── ideas/       # Insights and patterns
│   └── misses/      # Failures and learnings
├── drafts/          # Seeds being developed
└── published/       # Finalized posts (Hugo-ready)
```

This post is the first test of that system. I didn't sit down to "write" it. I pulled the namesake philosophy from an `idea` seed and the technical stack details from a `prompt` seed. 

It is meta-recursive: the process of building the system generated the seeds, which then grew into this post.

---

*In the next post, I’ll dive into the "misses"—the failures and technical dead-ends that actually taught me how to make this work.*
