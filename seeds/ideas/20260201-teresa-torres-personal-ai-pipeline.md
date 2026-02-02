---
date: 2026-02-01
session: ses_3e5ce2650ffezITFBkVK17PzIN
tags: [workflow, automation, productivity, morning-routine, ai-assistant]
stage: seed
content_potential: high
---

## The Idea

Build a "Teresa Torres-style /today command" for personal productivity - a single command that aggregates calendar, emails, and newsletters into a daily briefing one-pager in Obsidian, then git pushes for multi-machine sync.

## Context

While testing moltbot's morning-briefing skill, user referenced a maily.so article about Teresa Torres (product coach, non-developer) who built her own `/today` command with Claude Code. The article inspired reimagining the morning briefing as a complete personal AI pipeline.

## The Vision

```
📅 apple-calendar ──┐
📧 himalaya email ──┼──▶ 🌅 morning-briefing ──▶ 📝 Obsidian ──▶ git push
📰 newsletter-parser┘       (Korean one-pager)      (vault)       (sync)
```

Key insight from Teresa Torres article:
> "When I have a task, do I need to be involved? Can Claude just do it? Or does it need my input?"

## Raw Exchange

> User: "yes but output is not slack. it's one pager in obsidian vault. once you post morning brief you push to git so I can sync in any other machine."

## What Makes This Novel

1. **Non-Slack output** - Document-first, not chat-first
2. **Git sync** - Multi-machine workflow, not single-device
3. **Korean language** - Localized for user's preference
4. **Obsidian integration** - Knowledge management, not just notifications

## Expansion Potential

- Blog: "Building Your Own /today Command: A Teresa Torres Approach"
- Tutorial: Personal AI assistant for knowledge workers
- Comparison: Chat-based vs document-based AI workflows
