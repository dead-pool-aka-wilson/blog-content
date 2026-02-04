---
title: "The Multi-Feature Request: Batching Work for AI Efficiency"
date: 2026-02-22
author: Raoul
categories: [ai-prompting]
tags: [ultrawork, multi-feature, git-workflow, plugin-development]
draft: false
summary: "Four features, one prompt, zero clarifications. How to structure multi-feature requests that don't need hand-holding."
lang: en
cover:
  image: "/covers/cover-ultrawork.png"
---

> "add checking pr review and commit limit per pr feature. add prompt information in issue and add prompt and response information in pr. ultrawork"

One sentence. Four distinct features. Thirty-five minutes later: all implemented, tested, and documented.

No clarification questions. No scope discussion. No "did you mean this or that?" The prompt was dense but unambiguous.

## The Prompt

Let me break it down:

```
add checking pr review             → Feature 1: PR review checking
and commit limit per pr feature    → Feature 2: Commit limit enforcement
add prompt information in issue    → Feature 3: Enhanced issue templates
and add prompt and response        → Feature 4: PR context enrichment
information in pr.
ultrawork                          → Mode trigger: plan-first execution
```

Four features packed into 25 words, plus a mode modifier.

## Why It Worked

### 1. Independent Features

Each feature was self-contained:
- **PR review checking**: Block merge if not approved
- **Commit limit**: Block PR if too many commits
- **Issue enhancement**: Add prompt history to issues
- **PR context**: Add conversation context to PRs

None depended on another. They could be implemented in any order, tested separately, and even shipped independently.

### 2. Implicit Scope Clarity

"checking pr review" isn't detailed, but it's specific enough:
- It's about PR reviews (not code reviews, not design reviews)
- It's about checking their status (not requesting them)
- In the context of a Git workflow tool, the scope is obvious

The prompt relied on shared context. The AI knew we were enhancing a Git workflow plugin. "PR review" in that context has a clear meaning.

### 3. Mode Trigger

`ultrawork` activated a specific execution mode:
- Plan agent invocation before implementation
- Parallel task execution where possible
- Full verification before claiming completion
- No shortcuts or partial implementations

This single word changed the entire interaction pattern.

### 4. No Over-Specification

The prompt didn't specify:
- Implementation approach
- File locations
- API choices
- Error handling details

This left room for the AI to apply appropriate patterns from the existing codebase. Over-specification would have constrained good design.

## The Outcome

| Metric | Result |
|--------|--------|
| Features | 4 of 4 implemented |
| Time | ~35 minutes |
| Commits | 4 (one per feature) |
| Lines added | ~255 |
| Clarifications needed | 0 |

The plan agent broke the work into waves, identified dependencies, and executed features in parallel where possible.

## The Pattern

Effective multi-feature prompts follow this structure:

```
{feature1} + {feature2} + {feature3}. {mode}
```

Where:
- Features are independent and clearly bounded
- Features share context (same codebase, same domain)
- Mode modifier sets execution expectations

### When to Batch Features

Batch when features are:
- **Related**: Same system, shared context
- **Independent**: Can be implemented separately
- **Small-to-medium**: Each is a focused change
- **Well-understood**: No ambiguity in intent

Don't batch when features:
- Are interdependent (Feature 2 requires Feature 1's output)
- Need different technical approaches
- Have unclear scope
- Require significant discussion each

### Contrast: Bad Multi-Feature Prompt

```
add some checks for prs and issues, also improve the ux somehow, and make it faster. ultrawork
```

Problems:
- "some checks" - which ones?
- "improve ux somehow" - no specific change
- "make it faster" - which part? how much?

This would require clarification rounds for each vague element.

## The Lesson

Multi-feature requests work when you've already done the thinking about what you want. The prompt I used was concise because:

1. I knew the four features I needed
2. Each feature had clear boundaries
3. The context (Git workflow plugin) was established
4. The mode trigger set execution expectations

If you can't state your features concisely, you probably need more thinking before prompting. The AI can execute well-defined work efficiently. It can't read minds about vague intentions.

## The Takeaway

One sentence requested four features. All four shipped in 35 minutes without clarification.

The prompt was dense but precise:
- Four independent features
- Shared context (Git workflow)
- Clear scope per feature
- Mode trigger for execution style

When you know what you want, say it directly. Batch related work. Trust the AI to figure out implementation details. The fewer words you use, the less room for misinterpretation.

`ultrawork` indeed.
