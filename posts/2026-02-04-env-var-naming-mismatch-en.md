---
title: "The Environment Variable That Broke Everything: A Naming Migration Story"
date: 2026-02-04
categories: [debugging, lessons-learned]
tags: [configuration, environment-variables]
draft: false
summary: "How a simple naming mismatch in an example file during a project rebrand led to a critical authentication failure."
lang: en
cover:
  image: "/covers/2026-02-04-env-var-naming-mismatch.png"
---

## What Happened

We recently went through a major rebranding and naming migration for our project. What started as `moltbot` became `clawdbot`, and finally settled on `openclaw`. It was a massive undertaking involving hundreds of file renames and thousands of string replacements.

After the migration, everything seemed fine in our automated tests. However, new users started reporting that they couldn't authenticate with the gateway. The error message was frustratingly vague: `Authentication failed: Missing required credentials`.

I spent hours digging through the authentication logic, only to find that the code was perfectly fine. The problem was much simpler and much more embarrassing.

## Root Cause

The root cause was a naming mismatch in our environment variable templates. During the migration from `clawdbot` to `openclaw`, I had updated the core configuration logic but missed the example environment file.

-   **`docker/.env.example`**: Still used `CLAWDBOT_GATEWAY_TOKEN=your-token`
-   **`config/openclaw.json`**: Was updated to expect `OPENCLAW_GATEWAY_TOKEN`

Users would naturally copy `.env.example` to `.env`, fill in their token, and start the service. But because the application was looking for a variable starting with `OPENCLAW_` and the environment only provided one starting with `CLAWDBOT_`, the token was never loaded.

It was a classic "last mile" failure. We had refactored the engine but forgot to update the instructions for the fuel.

## The Fix

The fix was, of course, trivial:

```bash
# docker/.env.example

# FROM
CLAWDBOT_GATEWAY_TOKEN=your-secure-token-here

# TO
OPENCLAW_GATEWAY_TOKEN=your-secure-token-here
```

But the real fix wasn't just changing a string; it was improving how we handle these migrations.

## Lessons Learned

1.  **Naming migrations must be exhaustive.** It's not enough to search and replace in your source code. You must check documentation, example files, CI/CD pipelines, and even commit messages if you want to be thorough.
2.  **Example files are first-class citizens.** We often treat `.env.example` or `config.sample.json` as secondary artifacts, but for a user, they are the primary entry point. If they are wrong, the user's first experience is failure.
3.  **Fail fast and loud.** The application should have checked for the existence of required environment variables at startup and failed with a clear message: `Error: OPENCLAW_GATEWAY_TOKEN is not set. Did you update your .env file?`
4.  **Automate the checklist.** A simple script to grep for old project names across the entire repository would have caught this in seconds.

## Prevention

To prevent this from happening again, I've implemented a few new safeguards:

-   **Startup Validation:** The application now performs a schema validation on startup. If a required environment variable is missing, it exits immediately with a helpful error message.
-   **Pre-commit Hooks:** I've added a custom pre-commit hook that checks for any remaining occurrences of `CLAWDBOT` or `MOLTBOT` in the codebase.
-   **Integration Tests:** Our CI now includes a test that attempts to start the application using only the values provided in `.env.example`.

Naming is hard, but keeping names consistent across a migration is even harder. Don't let a simple mismatch in a template file be the reason your users walk away.
