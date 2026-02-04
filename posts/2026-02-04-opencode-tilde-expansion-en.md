---
title: "The Tilde Trap: Why OpenCode Configs Need Absolute Paths"
date: 2026-02-04
categories: [debugging, lessons-learned]
tags: [opencode, configuration, paths]
draft: false
summary: "How assuming shell-like tilde expansion in a JSON config led to a plugin loading failure and what it taught me about path handling."
lang: en
cover:
  image: "/covers/2026-02-04-opencode-tilde-expansion.png"
---

## What Happened

I was setting up a custom local plugin for OpenCode. To keep my configuration clean and portable (or so I thought), I used the tilde (`~`) to represent my home directory in the plugin path.

My `opencode.json` looked like this:
```json
{
  "plugins": [
    "~/.local/share/oh-my-opencode"
  ]
}
```

When I tried to start OpenCode, it failed spectacularly with a `BunInstallFailedError`. The error message was confusing:

```json
{
  "name": "BunInstallFailedError",
  "data": {
    "pkg": "~/.local/share/oh-my-opencode",
    "version": "latest"
  }
}
```

Why was it trying to "install" a local path as if it were an npm package?

## Root Cause

The root cause is that **OpenCode does not perform shell expansion on configuration paths.**

In a terminal, your shell (bash, zsh, etc.) automatically expands `~` to your full home directory path (e.g., `/Users/koed`). However, when a program reads a JSON file, it sees the literal string `"~/.local/share/oh-my-opencode"`.

Because OpenCode didn't recognize this as a valid absolute path on the filesystem, it assumed it was a package name and handed it off to the package manager (Bun) to install. Bun, of course, couldn't find a package named `~/.local/share/oh-my-opencode` on the registry, leading to the install failure.

## The Fix

The fix was to replace the tilde with the full, absolute path to the plugin directory.

```json
// FROM
"plugins": ["~/.local/share/oh-my-opencode"]

// TO
"plugins": ["/Users/koed/.local/share/oh-my-opencode"]
```

Once I provided the absolute path, OpenCode correctly identified it as a local directory and loaded the plugin without trying to reach out to an external registry.

## Lessons Learned

1.  **Don't assume shell features in config files.** Unless the documentation explicitly states that it supports tilde expansion or environment variables, assume it treats strings literally.
2.  **Absolute paths are safer.** While relative paths can work in some contexts, absolute paths eliminate ambiguity, especially when the application might be started from different working directories.
3.  **Understand the fallback behavior.** In this case, the fallback behavior (treating an invalid path as a package name) made the error much more confusing than a simple "File not found."
4.  **JSON is just data.** JSON has no built-in concept of a "home directory" or "environment variables." It's just a collection of strings, numbers, and booleans.

## Prevention

To avoid this "tilde trap" in the future:
-   **Always use absolute paths** in configuration files that point to local resources.
-   **Check the logs carefully.** The fact that it was a `BunInstallFailedError` was the key clue that the path was being misinterpreted as a package.
-   **Use setup scripts.** If you need portability across different users/machines, use a setup script to generate the config file with the correct absolute paths for the current environment.

It's a small character that carries a lot of weight in the shell, but in a JSON config, it's just a literal squiggle that can break your entire setup.
