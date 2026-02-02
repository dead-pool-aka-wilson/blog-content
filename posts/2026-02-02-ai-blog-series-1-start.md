---
title: "Building a Blog with AI: The Content Seed System"
date: 2026-02-02
author: Raoul
categories: [meta]
tags: [ai-blog-series, opencode, blog, content-creation]
draft: false
summary: "Why I stopped letting my AI conversations evaporate and built a system to harvest them into blog posts."
series: ["Building a Blog with AI"]
---

How much of your best thinking is currently rotting in a chat history you'll never open again?

Every day, I spend hours talking to AI. We debug complex race conditions, debate architectural trade-offs, and sometimes wander into deep philosophical territory. It's some of my most productive thinking.

But until recently, it was all ephemeral. The moment I closed a session, those insights vanished. I found myself explaining the same context or solving the same edge cases twice because I couldn't recall the exact nuance of yesterday's breakthrough. It felt like a massive leak in my intellectual pipes.

## The Discovery: Conversations as Content

I realized I didn't need to "write" a blog in the traditional sense. I just needed to stop letting the work I was already doing evaporate. I gave the AI a specific directive:

> "in dev folder of koed I want to gather prompts I wrote, Idea I had, misses we made and write blog contents with those. for example let's say i am creating blog with you right now, and I want this to be an content at the same time."

I wanted a system that captured three things:
1. **Prompts**: Not just the text, but why they worked.
2. **Ideas**: The raw sparks that happen mid-coding.
3. **Misses**: The bugs, the failed assumptions, the "aha!" moments after an hour of frustration.

## The Solution: The Content Seed System

I considered building a standalone tool or a separate database, but those felt like "more work." I needed something that lived where I already worked. I decided to integrate it into my existing Obsidian vault, **BrainFucked**.

Here is the structure we settled on:

```text
BrainFucked/10-Blog/
├── seeds/
│   ├── prompts/     # Prompts that actually delivered
│   ├── ideas/       # Raw insights to be developed
│   └── misses/      # Failures and lessons learned
├── drafts/          # Posts in progress
└── published/       # Finalized content (symlinked to Hugo)
```

I chose Obsidian because it allows me to use **wiki-links** to connect a raw "miss" to a final "post." It turns a flat list of files into a web of thought.

## One Hub to Rule Them All

I briefly debated keeping the blog content in a separate repo, but that violated my core principle: **reduce friction.**

If I have to switch contexts to record an idea, I won't do it. By making `BrainFucked` the central hub for everything—code, thoughts, and blog seeds—I removed the barrier between working and documenting.

- **Everything is searchable** in one place.
- **Backups are unified.**
- **Connections are organic.**

## Reflection

This post itself is the first test of the system. I didn't sit down to "write" it from scratch. I pulled the initial prompt from `seeds/prompts/`, the "one hub" philosophy from `seeds/ideas/`, and even the memory of my failed script attempts from `seeds/misses/`.

It’s meta-recursive. The process of building the system generated the seeds, which then grew into this post.

## Moving Forward

It wasn't entirely seamless. I spent two hours fighting with relative symlink paths that refused to resolve in Hugo, and I nearly deleted my entire `static` folder with a rogue bash script.

In the next part, I’ll talk about those failures and why "misses" are often more valuable than the successes.

---

*This post was crafted in conversation with AI, harvesting the very seeds it helped plant.*
