---
name: session-to-blog
description: Convert OpenCode sessions into blog posts. Activate when user wants to document a coding session, create a tutorial from recent work, or generate a case study. 세션을 블로그 포스트로 변환. 코딩 세션 문서화, 튜토리얼 생성, 케이스 스터디 작성 시 활성화.
---

# Session to Blog Post Converter

OpenCode 세션을 분석하여 블로그 포스트를 생성합니다.

## When to Activate

- User says "이 세션을 블로그로" / "turn this session into a blog post"
- User wants to document coding work / 코딩 작업을 문서화하고 싶을 때
- User requests tutorial/case study from session
- User says "세션 분석해서 포스트 만들어줘"

## Session Analysis Workflow

### Step 1: Session Discovery

```
# List all sessions
session_list(limit=10)

# Search for specific topic
session_search(query="검색어")

# Get session metadata
session_info(session_id="ses_xxx")
```

### Step 2: Content Extraction

```
# Read full session messages
session_read(session_id="ses_xxx", limit=100)
```

**Extract from session:**
- Problem statement (문제 정의)
- Solution approach (해결 접근법)
- Key code snippets (핵심 코드)
- Tool calls and results (도구 호출 결과)
- Learnings & insights (배운 점)

### Step 3: Screenshot Capture

**Use playwright skill for visual documentation:**

```
/playwright - capture screenshots of:
- Code editor states
- Terminal outputs
- Browser results
- Architecture diagrams
```

**Save location:** `static/images/posts/YYYY-MM-DD-title/`

## Hugo Output Template

### Frontmatter

```yaml
---
title: "[EN Title] / [KO 제목]"
date: YYYY-MM-DD
categories: [case-study, tutorial, development-log]
tags: [relevant, tags, here]
draft: false
lang: ko  # or en
---
```

### Korean Post Structure

```markdown
## 개요
[문제 상황과 목표 설명]

## 문제 정의
[구체적인 문제 설명]

## 해결 과정

### 접근법 1
[시도한 내용과 결과]

### 최종 해결책
[성공한 방법]

## 핵심 코드
\`\`\`[language]
// 코드 스니펫
\`\`\`

## 스크린샷
![설명](../images/posts/YYYY-MM-DD-title/screenshot-1.png)

## 배운 점
- [인사이트 1]
- [인사이트 2]

## 결론
[요약 및 다음 단계]
```

### English Post Structure

```markdown
## Overview
[Problem context and goal]

## Problem Definition
[Specific problem description]

## Solution Journey

### Approach 1
[What was tried and results]

### Final Solution
[What worked]

## Key Code
\`\`\`[language]
// code snippet
\`\`\`

## Screenshots
![description](../images/posts/YYYY-MM-DD-title/screenshot-1.png)

## Lessons Learned
- [Insight 1]
- [Insight 2]

## Conclusion
[Summary and next steps]
```

## Content Categories

| Category | Korean | When to Use |
|----------|--------|-------------|
| `case-study` | 케이스 스터디 | Detailed problem-solution narrative |
| `tutorial` | 튜토리얼 | Step-by-step learning guide |
| `development-log` | 개발 로그 | Technical decisions and progress |
| `debugging` | 디버깅 | Problem investigation and fix |
| `LLM-mastery` | LLM 활용 | AI/LLM usage patterns |

## Checklist

- [ ] Session identified via session_list/search / 세션 식별 완료
- [ ] Session content read via session_read / 세션 내용 읽기 완료
- [ ] Key insights extracted / 핵심 인사이트 추출
- [ ] Screenshots captured via playwright / Playwright로 스크린샷 캡처
- [ ] Post generated in Hugo format / Hugo 형식으로 포스트 생성
- [ ] Language selected (ko/en) / 언어 선택됨
- [ ] Frontmatter complete with all fields / 프론트매터 완성
- [ ] Images saved to static/images/posts/ / 이미지 경로 저장

---

**Remember**: The goal is to turn raw coding sessions into polished, shareable content that others can learn from.
