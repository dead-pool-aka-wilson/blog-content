---
title: "From Seeds to Posts: The Transformation"
date: 2026-02-05
categories: [meta]
tags: [ai-blog-series, content-creation, writing-process, workflow]
draft: false
summary: "How raw content seeds become finished blog posts - the curation, development, and transformation process"
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-5.png"
  alt: "Film clapperboard with seedling and flower"
---

## The Harvest Is Just the Beginning

In the [previous post](/posts/2026-02-05-ai-blog-series-4-automation/), I explained the technical infrastructure - skills, plugins, MCP servers. But infrastructure is only half the story.

Seeds accumulate. Great. Now what?

A seed is not a post. It's raw material - a half-formed thought, a code snippet, a debugging story captured in the moment. Transformation is required.

## The Seed Review

Every few days, I look at what's accumulated:

```
seeds/
├── ideas/
│   ├── 20260201-meta-content-creation.md      ← high potential
│   ├── 20260201-keychain-wrapper-secrets.md   ← high potential
│   ├── 20260202-lenis-scrollsnap-conflict.md  ← medium potential
│   └── ...
├── prompts/
│   └── 20260201-content-seeds-system.md       ← already used
└── misses/
    ├── 20260201-symlink-wrong-path.md         ← already used
    ├── 20260201-bash-arithmetic-gotcha.md     ← already used
    └── ...
```

Each seed has a `content_potential` rating: high, medium, low. This was assigned at capture time, but it's often wrong. Ideas that seemed brilliant at 2 AM look pedestrian in daylight.

### Filtering Criteria

Not every seed becomes a post. Good candidates have:

**Universality**: Does it apply beyond my specific situation? A symlink pointing to the wrong user path is universal. A bug in my specific codebase isn't.

**Teach-ability**: Can I explain it clearly? Some ideas feel profound but resist articulation. They need more time to develop.

**Story**: Is there a narrative? Debugging journeys work well. "I configured X" doesn't.

**Freshness**: Has this been covered to death? Another "how to set up Hugo" post won't help anyone.

## The Development Process

Once a seed is selected, I move it to drafts:

```bash
mv seeds/ideas/20260201-keychain-wrapper-secrets.md drafts/
```

Then the expansion begins. A seed typically has:
- A one-line idea
- Some context
- Maybe a code snippet or quote

A post needs:
- An opening hook
- Structured sections
- Concrete examples
- A payoff

### AI-Assisted Expansion

This is where AI collaboration shines. I use the `/thought-dialogue` skill:

> "Let's develop the keychain wrapper idea. Start with why someone would care."

The AI asks questions:
- "What problem does this solve that ~/.zshrc doesn't?"
- "Who's the target reader - security-conscious devs or beginners?"
- "Do you want to include the full wrapper script or just the concept?"

Through dialogue, the seed expands. I'm not writing in isolation - I'm having a conversation that generates prose.

### The Skill Stack

Different skills for different phases:

| Skill | When to Use |
|-------|-------------|
| `/content-seed` | Passive capture (always on) |
| `/thought-dialogue` | Developing raw ideas |
| `/session-to-blog` | Converting work sessions directly |
| `/reading-digest` | Summarizing external sources |

The right skill at the right moment.

## From Draft to Published

A draft isn't published until:

1. **Structure check**: Does the post flow? Headers in logical order?
2. **Example check**: Every claim supported by code or concrete example?
3. **Frontmatter complete**: Hugo needs `title`, `date`, `categories`, `tags`, `summary`, `series`
4. **Translation done**: Korean and English versions for this blog

### The Translation Question

This blog supports Korean and English. Every post needs both.

I write in English first (for me, it's easier to think in English about technical topics). Then AI helps translate, but not literally:

> "Translate this post to Korean. Maintain the conversational tone. Don't use formal honorifics - 해체 style."

Translation is creative work, not mechanical. A phrase that works in English might need complete restructuring in Korean.

## The Publishing Flow

Once both versions are ready:

```
drafts/keychain-wrapper.md
    ↓ (move to published/)
published/2026-02-10-keychain-wrapper.md
published/2026-02-10-keychain-wrapper.ko.md
    ↓ (symlinked to Hugo)
content/posts/2026-02-10-keychain-wrapper.md
    ↓ (hugo build)
public/posts/2026-02-10-keychain-wrapper/index.html
```

The symlink from `content/posts/` to `BrainFucked/10-Blog/published/` means Obsidian and Hugo see the same files. Edit in one place, deploy from another.

## The Numbers

From the sessions so far:

| Metric | Count |
|--------|-------|
| Seeds captured | 21 |
| Used in posts | 7 |
| Conversion rate | 33% |
| Drafts in progress | 3 |

Not everything becomes a post. That's fine. Seeds that don't convert aren't wasted - they inform future thinking, provide snippets for other posts, or just sit until the right moment.

## What Makes This Work

The key insight: **lower the barrier at every stage**.

- **Capture barrier**: Near zero. AI harvests automatically.
- **Review barrier**: Low. Just scan titles, check potential ratings.
- **Development barrier**: Reduced. AI helps expand through dialogue.
- **Translation barrier**: Reduced. AI handles first pass.

Each step has friction removed. Not eliminated - creative work still requires effort - but reduced enough that content actually flows.

## The Meta-Example

This post you're reading went through the same process:

1. **Seed**: Note about the drafts/ → published/ workflow
2. **Development**: `/thought-dialogue` to expand on transformation
3. **Draft**: Written in harvest worktree
4. **Translation**: Pending
5. **Published**: Now

The system documents itself.

---

*This post exists because the barrier to writing it was low enough that I actually did it.*

---

## Series So Far

1. [Building a Blog with AI: The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Content Begets Content: Discovering Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [The Tools: Under the Hood of AI-Powered Blogging](/posts/2026-02-05-ai-blog-series-4-automation/)
5. [From Seeds to Posts: The Transformation](/posts/2026-02-06-ai-blog-series-5-transformation/)

*More to come...*
