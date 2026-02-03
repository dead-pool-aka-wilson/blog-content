---
title: "Turning Seeds into Posts"
date: 2026-02-06
author: Raoul
categories: [meta]
tags: [ai-blog-series, content-creation, writing-process, workflow]
draft: false
summary: "Why the gap between thought and article kills most ideas, and how dialogue with AI bridges it."
cover:
  image: "/covers/cover-2026-02-06-ai-blog-series-5-transformation.png"
series: ["Building a Blog with AI"]
---

### WHY: Where Ideas Go to Die

I have twenty-one "seeds" sitting in a folder. Most are cryptic notes from 3 AM or orphan code snippets. In a traditional workflow, these would be the cemetery of "maybe someday." 

Ideas don't fail because they are bad; they fail because the friction between a raw thought and a finished article is too high. Most blogging systems expect you to bridge this gap with pure willpower—sitting down to "write" from scratch. 

I’ve realized that dialogue beats drafting. Friction reduction isn't just about speed; it's about survival. Ideas die in the silence of a blank page. They live when you have a sparring partner to pull the structure out of the mess.

### HOW: The Frictionless Pipeline

To bridge this gap, I’ve built a pipeline that treats writing as a series of low-energy conversions rather than a single monumental effort.

#### 1. AI as a Sparring Partner
Instead of staring at a cursor, I use the `/thought-dialogue` skill. I don't ask the AI to write for me; I ask it to argue with me. 
- "Why would a senior dev care about this?"
- "What's the one thing I'm missing here?"

This conversation turns a one-line seed into a structured argument. I am narrating my path through an idea rather than constructing it.

#### 2. Specialized Skills
Different phases of development require different types of intelligence. I use a dedicated set of skills to handle the heavy lifting:

| Skill | When to Use |
|-------|-------------|
| `/content-seed` | Passive capture during work |
| `/thought-dialogue` | Developing raw ideas into arguments |
| `/session-to-blog` | Converting technical logs into prose |
| `/reading-digest` | Synthesizing external research |

#### 3. Infrastructure: The Obsidian-Hugo Symlink
The final piece of "HOW" is removing the logistics. My Obsidian vault is symlinked directly to the Hugo `content/` directory. 
```bash
mv seeds/ideas/keychain-wrapper.md drafts/
# ... develop through dialogue ...
mv drafts/keychain-wrapper.md published/
```
There is no copy-pasting, no version mismatch, and no "uploading" process. Editing in my notes *is* preparing for deployment.

### WHAT: The Conversion Metrics

The effectiveness of this system is visible in the data. By lowering the barrier to entry at every stage, I’ve managed to maintain a consistent output that would have been impossible before.

| Metric | Count |
|--------|-------|
| Seeds captured | 21 |
| Used in posts | 7 |
| **Conversion rate** | **33%** |

In any other system, that conversion rate would be closer to 5%. This post exists because the effort required to turn the "workflow" seed into this article was lower than my urge to procrastinate.

### Closure: The End of the Beginning

This series was more than just a documentation of a blog setup. It was an experiment in meta-recursion: using AI to build a system for using AI to write about using AI. 

The friction is gone. The seeds are planted. Now, it’s just about tending the garden.

---

### Series Finale

1. [Building a Blog with AI: The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Content Begets Content: Discovering Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [The Tools: Under the Hood of AI-Powered Blogging](/posts/2026-02-05-ai-blog-series-4-automation/)
5. [Turning Seeds into Posts](/posts/2026-02-06-ai-blog-series-5-transformation/)
