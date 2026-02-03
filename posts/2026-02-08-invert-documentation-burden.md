---
title: "Invert the Documentation Burden: Let AI Harvest While You Think"
date: 2026-02-08
author: Raoul
categories: [meta]
tags: [workflow, AI, automation, content-creation]
draft: false
summary: "Stop documenting your work manually. Configure AI to silently capture insights, decisions, and mistakes while you focus on actual thinking."
cover:
  image: "/covers/cover-2026-02-08-invert-documentation-burden.png"
---

Every knowledge worker has the same problem: the best insights happen when you're too busy to write them down.

You're deep in a debugging session, and you realize something important about the architecture. You're in a meeting, and someone says something that reframes the whole project. You're reading code, and a pattern clicks that you've been missing for months.

By the time you have a moment to document these insights, they've faded. The context is gone. The urgency has passed. So you don't write them down, and the knowledge evaporates.

## WHY: The Capture Paradox

Documentation tools assume you'll interrupt your flow to capture information. "Just jot it down." "Add a quick note." "Tag it for later."

But the moments worth capturing are precisely the moments when you can't afford to context-switch. You're in flow. You're thinking. The act of pausing to document destroys the state that produced the insight in the first place.

I built a content seed system with manual capture commands: `seed-idea`, `seed-prompt`, `seed-miss`. The commands worked fine. The problem was me—I never remembered to use them when I was actually having insights. By the time I thought "I should capture this," I was already onto the next thing.

The capture paradox: valuable insights happen during high-context states, but documentation requires exiting those states.

## HOW: Passive Harvesting

The solution isn't better capture tools. It's removing capture from the human workflow entirely.

When I work with an AI assistant, the conversation already contains everything worth capturing:
- The problems I'm trying to solve
- The approaches I consider and reject
- The decisions I make and why
- The mistakes I encounter and how I fix them

All of this is externalized in the conversation. The AI doesn't need me to explicitly flag "this is an insight"—it can recognize patterns worth capturing and extract them silently.

The shift: from "user documents with AI help" to "AI documents while user works."

```
Traditional flow:
User works → User notices insight → User stops → User documents → User resumes

Passive harvesting:
User works → AI silently captures → User continues uninterrupted
```

I configured my AI assistant to monitor for three categories:

**Prompts**: Instructions I write that produce good results (or instructively bad results). These become prompt engineering examples.

**Ideas**: Insights that emerge during conversation—"aha" moments, pattern recognitions, novel approaches. These become potential blog posts or design principles.

**Misses**: Mistakes, bugs, wrong assumptions. These become debugging guides or cautionary tales.

The AI captures seeds into categorized files without asking permission. I review them later, develop the promising ones, and discard the noise.

## WHAT: Beyond Content Seeds

This pattern applies wherever humans generate information as a byproduct of work:

**Meeting notes**: Instead of someone taking notes (splitting their attention), let AI listen and summarize. The human participants stay fully engaged; the documentation happens passively.

**Code documentation**: Instead of writing docstrings after the fact, let AI observe the development conversation and generate docs from the context. "What were you trying to do here?" becomes unnecessary—the AI already knows.

**Decision logs**: Organizations lose institutional knowledge because decisions aren't recorded. But decisions are discussed—in Slack, in meetings, in emails. Passive harvesting can capture the rationale without requiring anyone to maintain a decision log.

**Learning journals**: Developers learn constantly but rarely record what they learn. An AI that monitors work sessions can extract lessons automatically, building a personal knowledge base from daily activity.

### The Trust Calibration

Passive harvesting requires trusting AI judgment about what's worth capturing. Early versions of my system over-captured—every minor observation became a seed, creating noise that was worse than nothing.

The calibration that works for me:
- **High bar for ideas**: Only capture if it could become a blog post or influence future decisions
- **Medium bar for prompts**: Capture if the pattern is reusable or the failure is instructive
- **Low bar for misses**: Capture most mistakes, because they're easy to discard but painful to forget

Your calibration will differ based on what you're optimizing for. The point is: define the criteria explicitly, then let AI apply them consistently.

### What Stays Human

Passive harvesting captures raw material. It doesn't replace judgment about what to develop or publish. I still review seeds weekly, decide which ones are worth expanding, and do the actual writing.

The AI handles the mechanical capture that I was failing to do consistently. The human handles the curation and refinement that requires taste and context.

This division of labor matches actual capabilities. AI is good at consistent application of defined criteria. Humans are good at deciding what matters and why.

## The Takeaway

The best documentation system is one you don't have to remember to use. Manual capture fails because it competes with the cognitive work that produces insights.

Invert the burden. Configure AI to harvest while you think. Let the documentation happen passively, as a byproduct of work, not an interruption to it.

My original quote to the AI was blunt: "seed idea capture and naming should be automated. I am just doing ideation and feedback on agent work and you harvest all."

That's the whole philosophy. You ideate. AI harvests. Knowledge compounds without friction.

What insights have you lost because you were too busy having them to write them down?
