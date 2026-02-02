---
date: 2026-02-01
project: personal-blog
severity: moderate
resolved: true
tags: [symlink, path, migration, hugo]
content_potential: medium
---

## What Happened

Hugo blog had a symlink `content/posts` pointing to `/Users/deok/brainFucked/10-Blog/published` - but the current user is `koed`, not `deok`. 

The blog appeared set up but couldn't find any content. The path was a remnant from a different machine/user setup.

## Why It Matters

Common gotcha when:
- Migrating dotfiles/configs between machines
- Cloning repos with absolute path symlinks
- Working across different user accounts

Symlinks with hardcoded absolute paths don't survive machine migrations.

## Resolution

```bash
rm content/posts
ln -s /Users/koed/Dev/BrainFucked/10-Blog/published content/posts
```

Also had to create the `published/` directory which didn't exist.

## Lesson

- Prefer relative symlinks when possible
- Check symlink targets after cloning repos
- Audit absolute paths in dotfiles before migration
