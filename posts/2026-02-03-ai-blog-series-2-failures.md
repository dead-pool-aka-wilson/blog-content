---
title: "Learning from Failure: The Value of Small Mistakes"
date: 2026-02-03
author: Raoul
categories: [meta]
tags: [ai-blog-series, debugging, bash, symlink, lessons-learned]
draft: false
summary: "Why I log my mistakes, and three concrete failures from building this blog."
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-2.png"
  alt: "Film clapperboard with X marks"
---

Failure is the fastest way to learn—but only if you bother to record it.

## Why: Documenting the "Mud"

Most of us try to hide our mistakes. We fix the bug, delete the messy logs, and pretend the solution was obvious all along. But I’ve found that documenting failure is actually more valuable than documenting success.

I log my mistakes for three reasons:
1. **To stop the loops.** If I don’t write down why a script failed, I will likely fall into the same trap three months from now.
2. **To help others.** Sharing the "mud" I got stuck in allows someone else to walk around it.
3. **To keep it real.** Success stories are often polished and distant. Failures are raw, specific, and relatable. They make for better content because they show the actual work.

## How: The "Misses" Category

In [Post 1](/posts/2026-02-02-ai-blog-series-1-start/), I mentioned my "content seed" system. While most people use seeds for ideas or prompts, I dedicated a specific category to **misses**.

Every time I spend more than 30 minutes on a "simple" problem, or when I have a "huh?" moment that turns into a revelation, I create a miss seed. I don't wait until I'm finished; I capture the confusion while it's fresh. This category acts as a graveyard for bad assumptions and a goldmine for future lessons.

## What: Three Concrete Failures

Here are three specific "misses" that occurred while building this blog infrastructure.

### 1. The Symlink Trap (Absolute vs. Relative)
I moved my blog repository to a new machine, ran `hugo server`, and saw an empty site. The posts were gone.

I checked `content/posts/` and found a broken symlink:
```bash
ls -la content/posts
# lrwxr-xr-x  content/posts -> /Users/olduser/brainFucked/10-Blog/published
```

The symlink was tied to my old username. I had assumed that since the repository was the same, the paths would hold. I forgot that **absolute paths are tied to a specific machine's soul**.

**The Fix:** I updated the symlink using a relative path where possible, or at least corrected it for the new environment.
**The Lesson:** When something disappears, check the plumbing before debugging the code.

### 2. Bash's Silent Arithmetic Kill
I wrote a test script that used `set -e` to exit on errors. The script would mysteriously quit after the first successful test increment.

```bash
#!/bin/bash
set -e
TESTS_PASSED=0

log_success() {
    ((TESTS_PASSED++))  # The script dies here
}
```

In Bash, `((0++))` returns `0`. Because `0` is treated as a "failure" exit code, `set -e` dutifully killed the script.

**The Fix:** Switched to a safer increment that doesn't rely on return values: `TESTS_PASSED=$((TESTS_PASSED + 1))`.
**The Lesson:** `set -e` turns minor syntax quirks into landmines. Bash arithmetic is older and weirder than you think.

### 3. API Keys and the .zshrc Habit
My first instinct for storing my Gemini API key was `~/.zshrc`. But exporting keys in a shell config is like leaving your house keys under the mat—every process can see them.

**The Fix:** I moved the keys to the macOS Keychain and wrote a small wrapper script:
```bash
export GEMINI_API_KEY=$(security find-generic-password -a "$USER" -s "gemini-api-key" -w)
```
This ensures the key is only in memory during the specific command’s execution.

**The Lesson:** Don't settle for the "default" way if it feels insecure. Secure plumbing is worth the extra few lines of Bash.

---

Failure is only a waste if you don't keep the receipt. By logging these misses, I’ve turned three annoying afternoons into permanent knowledge.

### Next Up
Documenting these failures revealed a pattern: the process of building became the content itself. In the final post, I’ll explore this "meta-recursion" and how it changed my perspective on writing.

---

## Series: Building a Blog with AI
1. [The Start: Infrastructure as Content](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Discovery: The Meta-Recursion Loop](/posts/2026-02-04-ai-blog-series-3-meta/)
