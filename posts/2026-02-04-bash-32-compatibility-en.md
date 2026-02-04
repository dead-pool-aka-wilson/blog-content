---
title: "Bash 3.2: macOS's Secret Compatibility Trap"
date: 2026-02-04
categories: [debugging, lessons-learned]
tags: [bash, macos, compatibility]
draft: false
summary: "How assuming modern Bash features on macOS led to a script failure and what I learned about portability."
lang: en
cover:
  image: "/covers/2026-02-04-bash-32-compatibility.png"
---

## What Happened

I was working on a setup script for the `openclaw` project, designed to automate environment configuration. To keep things organized, I decided to use associative arrays—a feature I've used countless times in modern Linux environments.

The script worked perfectly on my Ubuntu dev machine. However, the moment I ran it on a colleague's MacBook, it crashed with a cryptic error:

```bash
./setup.sh: line 12: declare: -A: invalid option
```

Line 12 was where I initialized my configuration map:
```bash
declare -A CONFIG_MAP
```

## Root Cause

The root cause is a well-known but often forgotten quirk of macOS: **it ships with Bash 3.2 by default.**

Bash 3.2 was released in 2007. Associative arrays (`declare -A`) were introduced in Bash 4.0 (2009). Because Apple stopped updating Bash after version 3.2 due to the licensing shift from GPLv2 to GPLv3, every macOS version—even the latest Sequoia—still includes this nearly 20-year-old version of Bash as `/bin/bash`.

I had fallen into the "it works on my machine" trap by assuming that `#!/usr/bin/env bash` would always point to a modern version of the shell. On macOS, unless the user has manually installed a newer version via Homebrew and updated their PATH, it points to the ancient system default.

## The Fix

To fix this, I had to refactor the script to be compatible with Bash 3.2 (and ideally POSIX sh). I replaced the associative arrays with a combination of indexed arrays and naming conventions.

Instead of:
```bash
declare -A mymap
mymap["key"]="value"
echo ${mymap["key"]}
```

I used a more traditional approach:
```bash
# Using variable names as keys
CONFIG_KEY="value"
# Or using indexed arrays if order mattered
KEYS=("key1" "key2")
VALUES=("val1" "val2")
```

For more complex data, I moved the configuration to a separate flat file and used `grep` or `awk` to retrieve values, which is much more portable across different Unix-like systems.

## Lessons Learned

1.  **Never assume the Bash version on macOS.** If you are writing scripts intended for macOS users, assume Bash 3.2 or target `zsh` (which is the default shell on modern macOS and is much more up-to-date).
2.  **POSIX sh is your friend.** If you don't need advanced Bash features, writing pure POSIX-compliant shell scripts ensures the highest level of portability.
3.  **Runtime Version Checks.** If your script *requires* Bash 4+, add a check at the beginning to fail gracefully with a helpful message:
    ```bash
    if [[ ${BASH_VERSINFO[0]} -lt 4 ]]; then
      echo "Error: This script requires Bash 4.0 or higher."
      exit 1
    fi
    ```
4.  **ShellCheck is essential.** Use tools like ShellCheck to catch compatibility issues early. It can be configured to target specific shell versions.

## Prevention

Moving forward, I've updated my development workflow to include:
- Testing scripts on a clean macOS environment (or a container mimicking one).
- Defaulting to `zsh` for macOS-specific automation.
- Using Python or Go for complex logic where shell portability becomes a headache.

Compatibility isn't just about the OS; it's about the tools the OS provides. Don't let a 2007 shell break your 2026 automation.
