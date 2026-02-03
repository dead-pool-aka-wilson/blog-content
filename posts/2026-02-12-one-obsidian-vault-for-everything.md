---
title: "One Vault to Rule Them All: Monorepo-Style Knowledge Management"
date: 2026-02-12
author: Raoul
categories: [meta]
tags: [knowledge-management, obsidian, workflow, centralization]
draft: false
summary: "Stop fragmenting your notes across locations. A single Obsidian vault with everything—seeds, drafts, posts—creates connections you'd never see otherwise."
cover:
  image: "/covers/cover-2026-02-12-one-obsidian-vault-for-everything.png"
---

Where do you put a note that's both a project idea and a blog seed?

I kept hitting this question while building a content capture system. The seed started in a project folder. Then it needed to link to a blog draft. Then the draft referenced meeting notes. Before long, I had three copies of related content in three locations, none of them linked.

The fix was to stop separating things.

## WHY: Fragmentation Kills Knowledge Work

Knowledge fragmentation has hidden costs:

**Search friction**: Which folder did I put that note in? Was it a seed, a draft, or a project note? You end up searching multiple locations for the same concept.

**Link breakage**: Obsidian's wikilinks are powerful, but they don't work across vaults. A seed in one location can't reference a project doc in another.

**Duplicate efforts**: Without links, you forget what you've already captured. You re-research topics. You re-write similar content.

**Graph blindness**: Obsidian's graph view shows conceptual connections—but only within a single vault. Fragmented vaults show fragmented graphs.

The knowledge compound interest only works when everything can connect to everything else.

## HOW: The Single Hub

My solution: one vault called BrainFucked that contains everything.

```
BrainFucked/
├── 00-Inbox/           # Quick capture
├── 01-Projects/        # Active work
├── 02-Areas/           # Ongoing responsibilities
├── 10-Blog/
│   ├── seeds/          # Captured ideas
│   ├── drafts/         # Work in progress
│   └── published/      # Final posts
├── 20-Archive/         # Completed projects
└── 99-Templates/       # Reusable structures
```

Everything is in one place. Seeds link to projects. Drafts reference meeting notes. Published posts link to the seeds that spawned them.

### The Naming Scheme

Numbered prefixes keep folders sorted and distinct:
- `00-09`: Temporary and incoming
- `01-09`: Active workspaces
- `10-19`: Content production
- `20+`: Archive and reference
- `99`: Meta/templates

This isn't the only valid structure—the key is that everything lives in one searchable, linkable space.

### Links Over Folders

The folder structure is for bulk organization. The real connections happen through links and tags:

```markdown
<!-- A seed -->
## The Idea
Something about [[API key security]].

## Related
- [[moltbot setup]] - where this came up
- [[security practices]] - broader topic
- #blog/technical - content category
```

When I search for "API key," I find the seed, the project context, and the related notes. The graph shows how these concepts connect.

### Why Not Multiple Vaults?

Obsidian supports multiple vaults. Some people recommend separate vaults for work/personal/projects. But:

1. **Cross-vault links don't work** - A seed can't reference a work note
2. **Search requires vault switching** - "Where did I put that?" multiplies
3. **Graph is incomplete** - You only see connections within one vault
4. **Sync complexity** - Multiple vaults = multiple sync configs

The cost of fragmentation outweighs the organizational "cleanliness" of separation.

## WHAT: Making It Work

### Tags for Facets
Use tags when content belongs to multiple categories:
- `#seed` - captured raw material
- `#draft` - work in progress
- `#published` - finalized
- `#project/moltbot` - project context
- `#blog/technical` - content type

A single note can be a seed, related to moltbot, and destined for the technical blog. Tags express this without requiring it to live in multiple folders.

### Search Operators
Obsidian search is powerful in a single vault:
- `path:Blog/seeds` - only seeds
- `tag:#project/moltbot` - only moltbot-related
- `[[API key]]` - notes linking to "API key"

Multiple vaults force you to repeat these searches per vault.

### Daily Notes as Entry Point
My daily note links to whatever I'm working on. Over time, this creates a temporal index—I can trace when ideas emerged by following daily note links backward.

### When to Split
The only time I'd use separate vaults:
- **Client work with confidentiality** - Can't risk cross-contamination
- **Shared vault** - A team vault separate from personal
- **Sandbox** - Experimenting with plugins that might break things

For personal knowledge management? One vault.

## The Takeaway

I asked my AI assistant where to put content seeds. Its first suggestion was a separate `.content-seeds/` directory. I pushed back: "BrainFucked will be hub for every idea, content, post."

That redirect improved the entire system. Seeds link to projects. Drafts reference sources. Published posts trace back to their origins. The graph shows connections I never would have made if content lived in separate locations.

Knowledge work compounds through connections. Connections require adjacency. Put everything in one vault and let the links emerge.

The messiness is the point. Your brain doesn't have folders.
