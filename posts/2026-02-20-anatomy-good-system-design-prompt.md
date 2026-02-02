---
title: "Anatomy of a Good System Design Prompt"
date: 2026-02-20
author: Raoul
categories: [ai-prompting]
tags: [system-design, content-workflow, meta]
draft: false
summary: "One sentence launched an entire content capture system. Breaking down what made a vague-sounding prompt surprisingly effective."
---

> "in dev folder of koed I want to gather prompts I wrote, Idea I had, misses we made and write blog contents with those. for example let's say i am creating blog with you right now, and I want this to be an content at the same time."

This prompt doesn't look special. No structured format. No explicit requirements. Grammatically informal. But it produced a complete content capture system—directory structure, templates, automation, and documentation—without a single clarification question.

What made it work?

## The Prompt

Let me quote it again, because the simplicity is the point:

> "in dev folder of koed I want to gather prompts I wrote, Idea I had, misses we made and write blog contents with those. for example let's say i am creating blog with you right now, and I want this to be an content at the same time."

45 words. No bullet points. No specification document. Yet it contained everything needed.

## What Made It Work

### 1. Clear Categories

Three types of content, explicitly named:
- **Prompts** - Instructions written to AI
- **Ideas** - Insights and concepts
- **Misses** - Mistakes and failures

These categories are distinct and non-overlapping. The AI could immediately propose a directory structure with three folders.

### 2. Concrete Location

"in dev folder of koed" - A real filesystem path. Not "somewhere convenient" or "in a good place." A specific starting point.

This grounding prevented abstract discussion about where things *could* go. It said where things *will* go.

### 3. Meta-Awareness

"let's say i am creating blog with you right now, and I want this to be an content at the same time"

This is the key insight buried in casual phrasing. The conversation itself should be captured. The act of designing the system is also content for the system.

This meta-instruction:
- Clarified the immediate use case
- Provided a real example to work with
- Created a constraint (the system must handle this conversation)

### 4. Open Design Space

The prompt specified *what* but not *how*. No requirements for file formats, naming conventions, or automation. This left room for the AI to propose solutions based on best practices.

Overly specific prompts ("use YAML frontmatter with these exact fields...") would have constrained the design. Leaving it open enabled discovery.

## The Response Quality

The AI's response demonstrated understanding:

1. **Proposed before implementing** - Showed a directory structure and asked for confirmation
2. **Built on existing context** - Discovered existing blog skills and integrated with them
3. **Created real examples** - Not just templates, but actual first entries
4. **Documented while building** - The README for the system was created as part of building the system

No clarification questions were needed. The prompt was specific enough to act on, open enough to allow good design.

## The Pattern

Effective system design prompts balance specificity and openness:

| Element | This Prompt | Why It Works |
|---------|-------------|--------------|
| What to build | "gather prompts, ideas, misses" | Clear scope |
| Where to build | "dev folder of koed" | Grounded location |
| Why to build | "write blog contents" | Clear purpose |
| Example use case | "creating blog with you right now" | Immediate validation |
| How to build | (not specified) | Design flexibility |

The prompt answered "what, where, why" and left "how" to the implementer.

## When This Pattern Works

Use vague-but-grounded prompts when:
- You trust the implementer's (AI or human) judgment
- You want creative solutions, not spec compliance
- The domain has established patterns to draw from
- You can iterate on the result

Use detailed specifications when:
- Exact compatibility requirements exist
- Multiple implementers need coordination
- The solution must match existing systems precisely
- Iteration is expensive

## The Anti-Pattern

The opposite extreme—overspecified prompts—often produces worse results:

```
Create a content capture system with the following requirements:
- Directory structure: seeds/prompts, seeds/ideas, seeds/misses
- File format: Markdown with YAML frontmatter
- Frontmatter fields: date (ISO 8601), tags (array), content_potential (enum: low/medium/high)
- Naming convention: YYYYMMDD-slug.md
- Include templates for each type
- Create shell commands: seed-prompt, seed-idea, seed-miss
- ...
```

This prompt assumes you already know the right design. If you knew the right design, you wouldn't need help building it.

## The Takeaway

The prompt that launched this blog's content system was informal, short, and lacked structure. It worked because:

1. Categories were clear (prompts, ideas, misses)
2. Location was concrete (dev folder)
3. Purpose was stated (blog content)
4. A real example was embedded (this conversation)
5. Implementation was left open

When asking for system design help, provide enough context to understand the problem, then get out of the way. The best prompts invite solutions rather than dictating them.

Mine was 45 words. It built a system I'm still using.
