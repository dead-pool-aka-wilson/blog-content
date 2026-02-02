---
title: "Turning Seeds into Posts"
date: 2026-02-05
author: Raoul
categories: [meta]
tags: [ai-blog-series, content-creation, writing-process, workflow]
draft: false
summary: "How raw thoughts become finished articles by reducing the friction of development."
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-5.png"
  alt: "Film clapperboard with seedling and flower"
---

I have twenty-one "seeds" sitting in a folder. Most of them are incomprehensible notes I wrote at 3 AM, or code snippets without context. If I had to turn these into blog posts manually, I’d probably just delete the folder and go back to sleep.

The problem isn't having ideas. It's that the gap between a raw thought and a structured article is a valley of friction. Most blogging systems fail because they expect you to bridge that gap with pure willpower.

### The Problem with Potential

Every few days, I look at the pile:

```
seeds/
├── ideas/
│   ├── 20260201-meta-content-creation.md      ← high potential
│   ├── 20260201-keychain-wrapper-secrets.md   ← high potential
│   ├── 20260202-lenis-scrollsnap-conflict.md  ← medium potential
│   └── ...
```

When I first capture these, I give them a rating. But I've discovered that my 2 AM enthusiasm is a terrible judge of quality. What seemed like a "high potential" insight often looks like a generic observation in the morning.

The friction comes from the "Now what?" moment. A seed is just a hint. To make it a post, I used to think I had to sit down and "write." That's where I usually stopped.

### Discovery: AI is a Sparring Partner

I realized that the most useful thing AI can do isn't writing the draft for me. It's arguing with me. 

Instead of staring at a blank page, I use the `/thought-dialogue` skill. I tell the AI, "I want to talk about this keychain wrapper idea. Why would a smart dev care about this?"

The AI doesn't just generate text. It asks questions:
- "What problem does this solve that ~/.zshrc doesn't?"
- "Is this for beginners or security-conscious devs?"

This dialogue turns a one-line seed into a structured argument. I'm not writing in isolation; I'm having a conversation that generates prose. I'm narrating my path through the idea rather than trying to construct it from scratch.

### The Solution: A Frictionless Pipeline

Once the dialogue has some meat, the rest is just plumbing. Different skills handle different phases:

| Skill | When to Use |
|-------|-------------|
| `/content-seed` | Passive capture |
| `/thought-dialogue` | Developing raw ideas |
| `/session-to-blog` | Converting work sessions |
| `/reading-digest` | Summarizing sources |

The actual publishing is just moving files:

```bash
mv seeds/ideas/20260201-keychain-wrapper-secrets.md drafts/
# ... develop through dialogue ...
mv drafts/keychain-wrapper.md published/
```

Because my Obsidian vault is symlinked to the Hugo `content/` folder, editing in my note-taking app automatically prepares the post for deployment. I don't have to copy-paste or manage multiple versions.

### Reflection: Lowering the Barrier

The numbers tell the story:

| Metric | Count |
|--------|-------|
| Seeds captured | 21 |
| Used in posts | 7 |
| Conversion rate | 33% |

In a traditional workflow, that conversion rate would be much lower. By lowering the barrier at every stage—capture, development, and publishing—I actually get things done.

This post exists because the effort required to turn the "workflow" seed into this article was low enough that I didn't talk myself out of it.

---

### Series So Far

1. [Building a Blog with AI: The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Content Begets Content: Discovering Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [The Tools: Under the Hood of AI-Powered Blogging](/posts/2026-02-05-ai-blog-series-4-automation/)
5. [Turning Seeds into Posts](/posts/2026-02-06-ai-blog-series-5-transformation/)

*More to come...*
