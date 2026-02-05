---
title: "The Symlink That Pointed to Nobody"
date: 2026-02-14
author: Raoul
categories: [debugging]
tags: [symlink, path, migration, hugo]
draft: false
summary: "My Hugo blog showed zero posts. The content was there—just pointed at a user who doesn't exist on this machine."
lang: en
cover:
  image: "/covers/cover-2026-02-14-symlink-absolute-path-trap.png"
---

The blog was set up. Hugo ran without errors. But when I opened localhost:1313, the posts section was empty.

I knew there should be content. I'd written posts. The directory structure looked right. But Hugo saw nothing.

## The Symptom

```bash
hugo server
# Started successfully, no errors
# But: Posts: 0, Pages: 3

ls content/
# posts  skills  ...
# posts directory exists!

ls content/posts/
# ... nothing
```

The `posts` directory existed but was empty. Or was it?

## The Investigation

```bash
ls -la content/posts
# lrwxr-xr-x  content/posts -> /Users/olduser/projects/blog/published
```

A symlink. Pointing to `/Users/olduser/...`. 

But I'm not `olduser`. I'm `newuser`. This machine has no `/Users/olduser` directory.

The symlink was a ghost reference from a different machine, a different time, a different user account. Hugo followed the link, found nothing, and silently showed zero posts.

## Why This Happens

Symlinks with absolute paths are machine-specific. They encode:
- The full filesystem path
- The username (in home directory paths)
- The machine's directory structure

When you clone a repo or copy dotfiles to a new machine, these paths don't update. The symlink still points to the old location, which doesn't exist.

Common scenarios:
- **Dotfiles repos** - Configs often symlink to user-specific paths
- **Monorepo setups** - Symlinks between packages use absolute paths
- **Backup restores** - Old machine's paths embedded in symlinks
- **Docker volumes** - Host paths baked into container configs

## The Fix

```bash
# Remove broken symlink
rm content/posts

# Create new symlink with correct path
ln -s "$HOME/Dev/blog-content/published" content/posts

# Verify
ls content/posts/
# 2026-02-02-ai-blog-series-1.md  ...
```

Also had to create the target directory, which didn't exist on this machine:

```bash
mkdir -p "$HOME/Dev/blog-content/published"
```

### Diagnose Broken Symlinks

Quick commands to find and verify symlink issues:

```bash
# Find ALL symlinks in current directory
find . -type l -exec ls -la {} \;

# Find BROKEN symlinks specifically
find . -type l ! -exec test -e {} \; -print

# Check what a specific symlink points to
readlink -f content/posts

# Test if symlink target exists
test -e "$(readlink content/posts)" && echo "OK" || echo "BROKEN"
```

## Prevention Strategies

### Use Relative Symlinks
When possible, create symlinks relative to the link location:

```bash
# Instead of:
ln -s /Users/koed/Dev/BrainFucked/10-Blog/published content/posts

# Use:
cd content
ln -s ../../BrainFucked/10-Blog/published posts
```

Relative symlinks survive directory moves (as long as relative positions stay the same).

### Check Symlinks After Cloning
Add to your "new machine setup" checklist:

```bash
# Find all symlinks in repo
find . -type l -exec ls -la {} \;

# Check for broken symlinks
find . -type l ! -exec test -e {} \; -print
```

### Use Environment Variables in Scripts
If symlinks must be absolute, generate them from environment:

```bash
ln -s "${HOME}/Dev/BrainFucked/10-Blog/published" content/posts
```

This at least adapts to the current user.

### Document Path Dependencies
In your README, list any paths that need updating after clone:

```markdown
## Post-Clone Setup
Update symlinks for your machine:
- `content/posts` → Your `published/` directory
```

## The Broader Lesson

The bug was silent. No error, no warning. Hugo saw an empty directory and continued. I only noticed because I expected content that wasn't there.

Silent failures are the worst kind. A broken symlink doesn't throw an exception—it just resolves to nothing.

When something "disappears" after a machine migration:
1. Check symlinks first: `ls -la` shows the target
2. Verify targets exist: `test -e $(readlink file)`
3. Look for absolute paths: anything starting with `/Users/` or `/home/`

The plumbing failed before the code ever ran. Sometimes debugging isn't about code—it's about the filesystem underneath it.
