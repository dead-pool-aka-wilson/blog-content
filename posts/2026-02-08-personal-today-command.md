---
title: "Building Your Own /today Command"
date: 2026-02-08
author: Raoul
categories: [meta]
tags: [workflow, automation, productivity, morning-routine, ai-assistant]
draft: false
summary: "Inspired by Teresa Torres, I built a single command that pulls my calendar, email, and newsletters into a daily briefing document—then syncs it everywhere."
lang: en
cover:
  image: "/covers/cover-2026-02-08-personal-today-command.png"
---

What if checking your morning commitments took one command instead of opening five apps?

I came across an article about Teresa Torres—a product coach, not a developer—who built her own `/today` command using Claude Code. Her insight was simple but profound: "When I have a task, do I need to be involved? Can Claude just do it? Or does it need my input?"

That question reframed how I think about personal automation. Not "what can AI do for me?" but "what am I doing that doesn't actually need me?"

## WHY: The Morning Ritual Problem

Every morning, I check the same things:
- Calendar: What meetings do I have?
- Email: Anything urgent overnight?
- Newsletters: Any industry news worth knowing?

Each lives in a different app. Each requires context-switching. Each pulls me into a reactive mode where I'm responding to inboxes instead of setting my own priorities.

The information itself isn't the problem—I do need to know my schedule. The problem is the *gathering*. It's repetitive cognitive work that doesn't benefit from my attention. It's exactly the kind of task Torres identified: something that doesn't need me.

## HOW: Document-First, Not Chat-First

Most AI assistant setups dump output into Slack or a chat window. That works for quick lookups, but it doesn't fit how I actually work. My knowledge lives in Obsidian. My planning happens in markdown files. A morning briefing that disappears into a chat stream is useless by afternoon.

The design principle: output goes to a document, not a conversation.

```
📅 apple-calendar ──┐
📧 email inbox    ──┼──▶ morning-briefing ──▶ Obsidian ──▶ git push
📰 newsletters    ──┘     (one-pager)          (vault)      (sync)
```

One command triggers the pipeline. The output is a markdown file that lands in my Obsidian vault. Then it pushes to git so I can access it from any machine.

### The Components

**Calendar integration:** Pull today's events from Apple Calendar. Format them as a simple timeline.

**Email summary:** Use himalaya (a CLI email client) to fetch unread messages. Summarize the important ones, flag anything requiring action.

**Newsletter digest:** Parse saved newsletters, extract key insights. Most newsletters have 2-3 worthwhile points buried in filler.

**Document generation:** Combine everything into a single markdown page. Include a "focus priorities" section at the top that I fill in manually.

**Git sync:** Push the daily file to a repo. This lets me open my morning briefing on my laptop, phone, or any machine with git access.

### What the Output Looks Like

```markdown
# Morning Briefing - 2026-02-08

## Focus Priorities
- [ ] _Fill in your top 3 for today_

## Today's Schedule
- 10:00 - Team standup (30 min)
- 14:00 - Design review with Sarah
- 16:00 - 1:1 with manager

## Email Summary
**Needs response:**
- Client X asking about timeline (flagged urgent)

**FYI only:**
- Newsletter signup confirmation
- AWS billing notification

## News Digest
- [Interesting article title] - Key takeaway in one sentence
- [Another article] - Why it matters
```

The document is structured for skimming. I open it, spend 30 seconds scanning, fill in my priorities, and I'm oriented for the day. No app-hopping, no context-switching, no inbox rabbit holes.

## WHAT: Making It Yours

The specific tools don't matter. Torres used Claude Code. I use a combination of shell scripts and moltbot (my personal automation bot). You could use Shortcuts, n8n, or any automation platform.

What matters is the mental model:

1. **Identify repetitive cognitive work** - What do you do every day that doesn't benefit from your judgment?

2. **Design document-first** - Where should the output live? Make it somewhere you'll actually look.

3. **Build for sync** - Your automation is useless if it only works on one device.

4. **Keep humans in the loop for judgment** - The briefing doesn't tell me what to prioritize. It gives me the information; I make the decision.

### Iteration, Not Perfection

My first version was overengineered. It tried to categorize emails by project, summarize every newsletter, and generate action items automatically. Too much AI inference, not enough raw information.

The current version is simpler: aggregate, format, sync. That's it. I can scan raw email subjects faster than I can verify AI-generated summaries. The automation removes the gathering friction; the judgment stays with me.

### The Obsidian Advantage

Using Obsidian as the destination has benefits beyond sync:
- Daily notes link to project notes via wikilinks
- I can search across months of briefings
- Patterns emerge: "I've had three urgent emails from Client X this month"

The briefing isn't just a disposable checklist. It becomes part of my knowledge graph, connected to everything else I'm thinking about.

## The Takeaway

Torres's question cuts through the hype around AI assistants: "Does this task need my input, or can Claude just do it?"

Morning briefings don't need my input. Gathering calendar events doesn't need my judgment. Summarizing routine emails doesn't benefit from my attention.

Build a command for the tasks that don't need you. Save your attention for the ones that do.

My `/today` command runs while I make coffee. By the time I sit down, the briefing is waiting in Obsidian, synced to every device. The day starts with clarity instead of inbox anxiety.

What's your equivalent of checking five apps every morning?
