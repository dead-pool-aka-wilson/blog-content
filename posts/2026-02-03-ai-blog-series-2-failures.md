---
title: "Learning from Failure: The Value of Small Mistakes"
date: 2026-02-03
author: Raoul
categories: [meta]
tags: [ai-blog-series, debugging, bash, symlink, lessons-learned]
draft: false
summary: "Three concrete failures I encountered while building this blog and the lessons they left behind."
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-2.png"
  alt: "Film clapperboard with X marks"
---

Have you ever spent three hours debugging a problem, only to realize the solution was a single line you wrote six months ago? I do it more often than I'd like to admit.

In my [previous post](/posts/2026-02-02-ai-blog-series-1-start/), I introduced my content seed system. It has categories for prompts, ideas, and **misses**. That last one is where I record my failures.

Why bother logging mistakes? 

First, to stop the loops. If I don't write it down, I forget. Then three months later, I'll find myself digging the same hole. Second, it's for you. If I can share the mud I got stuck in, maybe you can walk around it. Finally, failure is just better content. Success stories are often polished and distant; failures are raw, specific, and relatable.

Here are three "misses" from the making of this blog.

## 1. The Symlink Trap

I cloned my blog repo, ran `hugo server`, and... nothing. The server was up, but the posts were gone.

I checked `content/posts/` and found an empty directory. I was confused. I had definitely set this up before. Then I checked the file details and saw the ghost:

```bash
ls -la content/posts
# lrwxr-xr-x  content/posts -> /Users/olduser/brainFucked/10-Blog/published
```

The problem was staring at me. My current user is `newuser`, but the symlink was still reaching for `/Users/olduser/`. I had moved to a new machine, restored my dotfiles, and assumed the paths would magically follow.

**Why I thought that way:**
I assumed that since the Git repository was the same, the environment would be identical. I forgot that **absolute paths are tied to a specific machine's soul**.

**The Fix:**
I updated the symlink to the correct path:
```bash
rm content/posts
ln -s /Users/newuser/Dev/BrainFucked/10-Blog/published content/posts
```

**The Lesson:**
Use relative paths for symlinks whenever possible. Also, when you see "nothing" where something should be, don't look for a bug in the code first—look at the plumbing.

## 2. Bash's Silent Kill

I wrote a simple test script. Every time a test passed, it would increment a counter. But the script would just quit after the very first success. No error, no warning. Just dead.

```bash
#!/bin/bash
set -e  # Exit immediately on error
TESTS_PASSED=0

log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++))  # The script dies right here
}

log_success "First test"
log_success "Second test"
```

**What happened:**
I used the arithmetic expression `((TESTS_PASSED++))`. In Bash, this expression returns the value *before* the increment. 

1. `TESTS_PASSED` starts at 0.
2. `((TESTS_PASSED++))` returns 0.
3. In Bash, a return value of 0 is considered "falsy" or an error code.
4. Because I had `set -e` enabled, Bash saw that "error" and dutifully killed the script.

**The Fix:**
Switch to a safer increment method that doesn't rely on the return value of the expression:
```bash
log_success() {
    echo "[PASS] $1"
    TESTS_PASSED=$((TESTS_PASSED + 1))
}
```

**The Lesson:**
`set -e` is great for catching unexpected errors, but it turns every minor syntax quirk into a landmine. Be especially careful with arithmetic in Bash; its logic is older and weirder than you think.

## 3. Where to Put the API Keys

When setting up OpenCode, I needed a place to store my Gemini API key. My first instinct was the classic `~/.zshrc`:

```bash
export GEMINI_API_KEY="sk-..."
```

But I felt uneasy. Exporting keys in a shell config is like leaving your house keys under the mat. Every single process you run can see them. They might end up in your shell history, and if you ever share your dotfiles, you might accidentally leak them to the world.

**The Alternative: The Keychain Wrapper**
Instead of a global export, I wrote a wrapper script that pulls the key from the macOS Keychain only when needed.

```bash
#!/bin/bash
# ~/bin/openclaw - A wrapper script

fetch_key() {
    local keychain_name="$1"
    security find-generic-password -a "$USER" -s "$keychain_name" -w 2>/dev/null || {
        echo "Error: $keychain_name not found in Keychain" >&2
        return 1
    }
}

export GEMINI_API_KEY=$(fetch_key "gemini-api-key") || exit 1
exec /path/to/actual/command "$@"
```

**Why this is better:**
The key is stored securely in the system Keychain. It's only loaded into memory during the execution of that specific command. Once the command finishes, the key vanishes from the environment.

**The Lesson:**
Don't settle for the "default" way if it feels insecure. A little bit of extra plumbing—like a wrapper script—can significantly reduce your security risk.

## Recording the Mud

These three mistakes have one thing in common: they were hard to find but easy to fix once discovered. 

If I hadn't recorded them in my `misses/` category, I probably would have made them again in six months. By turning them into content, I'm forced to reflect on **why** they happened, which is the only way to actually learn.

Failure is only a waste if you don't keep the receipt.

## Next Up

While documenting these failures, I noticed an interesting pattern. The process of building this blog was becoming the content itself. In the final post of this series, I'll talk about this "meta-recursion" and how it changed my perspective on writing.

---

## Series

1. [Building a Blog with AI: The Start](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Content Begets Content: The Discovery of Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
