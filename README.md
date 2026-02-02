# Blog Content

OpenCode 기반 블로그 콘텐츠 관리 저장소. Obsidian vault로 사용 가능.

## Structure

```
blog-content/
├── posts/           # Published blog posts (Hugo-ready)
├── drafts/          # Work in progress
├── seeds/           # Auto-harvested content seeds
│   ├── ideas/       # Insights and concepts
│   ├── prompts/     # Effective AI prompts
│   ├── misses/      # Failures and learnings
│   └── visuals/     # Memes and images
├── skills/          # Content creation skills for OpenCode
├── search.md        # Hugo search page
└── .obsidian/       # Obsidian vault config (minimal)
```

## Skills

| Skill | Purpose | Trigger |
|-------|---------|---------|
| `session-to-blog` | OpenCode 세션 → 블로그 포스트 | "세션을 블로그로" |
| `philosophical-interview` | 소크라틱 철학적 인터뷰 | "소크라틱 대화" |
| `reading-digest` | 읽은 자료 요약 + 의견 | "이 책 정리해줘" |
| `project-journal` | 프로젝트 상태 업데이트 | "프로젝트 업데이트" |
| `thought-dialogue` | 개인적 생각 대화형 탐구 | "생각 정리 도와줘" |
| `meme-adapt` | 밈/이미지 Gemini 수정 | "이 밈 수정해줘" |
| `content-scout` | brainFucked 노트 분석 → 블로그 추천 | "블로그 추천해줘" |

## Content Workflow

```
[Work with AI] → [Seeds auto-harvested]
       ↓
[Develop in drafts/]
       ↓
[Move to posts/] → [Hugo builds via hosting repo]
```

## Obsidian Setup

This repo is Obsidian-compatible. Open as vault:

```bash
# Clone
git clone https://github.com/dead-pool-aka-wilson/blog-content.git

# Open in Obsidian
# File → Open Vault → Select blog-content folder
```

Minimal `.obsidian/` config included. Customize as needed.

## Integration with Hosting

This repo is used as a git submodule in [personal-blog-hosting](https://github.com/dead-pool-aka-wilson/personal-blog-hosting):

```bash
# In hosting repo:
git submodule add https://github.com/dead-pool-aka-wilson/blog-content.git content
```

Hugo reads posts from `content/posts/`.

## Languages

- English (default)
- Korean (`.ko.md` suffix)

## License

Private content. All rights reserved.
