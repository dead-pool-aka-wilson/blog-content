# Content Seeds

Raw materials for blog content. **Auto-harvested by AI** during sessions.

## Location (BrainFucked Hub)

```
BrainFucked/10-Blog/
├── seeds/           # ← You are here
│   ├── prompts/     # Prompts written to AI
│   ├── ideas/       # Insights, patterns, concepts
│   ├── misses/      # Failures, mistakes, learnings
│   └── visuals/     # Memes, diagrams, images
│       ├── sources/   # Original collected memes
│       └── adapted/   # Gemini-modified versions
├── drafts/          # Seeds being developed
└── published/       # Ready for Hugo (symlinked)
```

## Workflow

```
[AI auto-harvests] → seeds/
        ↓
[develop] → drafts/
        ↓
[publish] → published/ → Hugo blog
```

## How It Works

1. **You work** - ideate, give feedback, make decisions
2. **AI harvests** - automatically captures seeds during sessions
3. **Review weekly** - scan `seeds/` for content worth developing
4. **Develop** - move promising seeds to `drafts/`, expand
5. **Publish** - move finished posts to `published/`

## No Manual Capture Needed

The AI proactively monitors for:
- Notable prompts (good or instructive failures)
- Insights and "aha" moments
- Mistakes with learning value
- Design decisions with rationale

Seeds are auto-named based on content:
```
20260201-user-ideates-ai-harvests.md
20260201-symlink-wrong-user-path.md
```

## Shell Commands (if manual needed)

```bash
seed-prompt "title"   # Capture a prompt
seed-idea "title"     # Capture an idea
seed-miss "title"     # Capture a mistake
seed-list             # See recent seeds
```

## Visuals Workflow

### Sourcing (your job)
- Save memes from Twitter/TikTok/Reddit to `visuals/sources/`
- Create `.meta.md` sidecar with source URL, tags, context

### Adapting (via Gemini)
- Use `/meme-adapt` skill
- Save to `visuals/adapted/`
- Create metadata linking to source

## Obsidian Integration

Seeds are markdown files - fully Obsidian-compatible:
- Tag-based organization via YAML frontmatter
- Link between seeds using `[[wikilinks]]`
- Search across all seeds
- Graph view shows connections

## Tips

- AI captures, you review
- One seed per file
- File names: `YYYYMMDD-topic-slug.md`
- Link related seeds via tags
- Meta-content welcome
