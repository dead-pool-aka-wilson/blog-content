---
name: reading-digest
description: Create blog posts from reading materials with summaries and personal opinions. Activate when user wants to document book notes, article reviews, or paper summaries. 읽은 자료를 요약하고 의견을 담은 블로그 포스트 생성. 책 정리, 기사 리뷰, 논문 요약 시 활성화.
---

# Reading Digest Generator

읽은 자료(책, 기사, 논문, 영상)를 분석하고 개인 의견을 담은 블로그 포스트를 생성합니다.

## When to Activate

- User shares reading material (book, article, paper)
- User says "이 책 정리해줘" / "summarize this reading"
- User wants to create a review post
- User shares quotes for commentary / 인용문에 대한 코멘터리 요청
- User says "읽은 것 블로그로 만들어줘"

## Content Types

### Books (책)
- Chapter-by-chapter summaries (장별 요약)
- Key concepts and frameworks (핵심 개념과 프레임워크)
- Memorable quotes (기억에 남는 구절)
- Personal takeaways (개인적 교훈)

### Articles/Papers (기사/논문)
- Main thesis identification (핵심 논지 파악)
- Supporting arguments analysis (뒷받침 논거 분석)
- Methodology critique (방법론 비평) - for papers
- Critical analysis (비판적 분석)

### Videos/Podcasts (영상/팟캐스트)
- Key points with timestamps (타임스탬프와 핵심 포인트)
- Speaker's main arguments (발표자의 주요 주장)
- Notable quotes/moments (주목할 만한 순간)
- Personal reflections (개인적 성찰)

## Analysis Framework

### Core Questions to Answer

| Question | Korean | Purpose |
|----------|--------|---------|
| **What?** | 무엇? | What is the main argument/thesis? |
| **So What?** | 그래서? | Why does this matter? |
| **Now What?** | 그래서 어쩌지? | How does this apply to me/us? |

### Reading Lens Options

| Lens | Korean | Focus |
|------|--------|-------|
| **Analytical** | 분석적 | Logical structure, evidence quality |
| **Personal** | 개인적 | Resonance, disagreements, connections |
| **Practical** | 실용적 | Applications, action items |
| **Critical** | 비판적 | Weaknesses, blind spots, counterarguments |

## Hugo Output Template

### Frontmatter

```yaml
---
title: "Book Review: [Title] / 북 리뷰: [제목]"
date: YYYY-MM-DD
categories: [book-review, reading-notes, article-review]
tags: [author-name, topic-tags]
draft: false
lang: ko  # or en
source:
  title: "[Original Title]"
  author: "[Author Name]"
  year: YYYY
  type: book/article/paper/video
  url: "[URL if applicable]"
---
```

### Korean Post Structure

```markdown
## 기본 정보
- **제목:** [원제목]
- **저자:** [저자명]
- **출판:** [출판사, 연도]
- **읽은 날짜:** YYYY-MM-DD

## 한 문장 요약
[핵심 메시지를 한 문장으로]

## 핵심 내용

### 주요 논점
1. [저자의 핵심 주장 1]
2. [저자의 핵심 주장 2]
3. [저자의 핵심 주장 3]

### 인상적인 구절

> "[인용문]" (p. XX)

[이 구절에 대한 나의 생각]

> "[또 다른 인용문]" (p. XX)

[이 구절에 대한 나의 생각]

## 나의 의견

### 공감한 부분
[동의하는 내용과 이유]

### 의문이 드는 부분
[비판적 시각이나 질문]

### 연결되는 생각들
[다른 책/아이디어와의 연결]

## 적용점
- [실제 삶/일에 적용할 수 있는 것 1]
- [실제 삶/일에 적용할 수 있는 것 2]

## 평점

⭐⭐⭐⭐☆ (4/5)

[한 줄 평가]

## 추천 대상
[이 책/자료가 도움이 될 사람]
```

### English Post Structure

```markdown
## Overview
- **Title:** [Original Title]
- **Author:** [Author Name]
- **Published:** [Publisher, Year]
- **Read on:** YYYY-MM-DD

## One-Sentence Summary
[Core message in one sentence]

## Key Ideas

### Main Arguments
1. [Author's key claim 1]
2. [Author's key claim 2]
3. [Author's key claim 3]

### Notable Quotes

> "[Quote]" (p. XX)

[My thoughts on this quote]

> "[Another quote]" (p. XX)

[My thoughts on this quote]

## My Take

### What Resonated
[Points of agreement and why]

### Questions & Critiques
[Critical perspectives or questions]

### Connections
[Links to other books/ideas]

## Takeaways
- [Practical application 1]
- [Practical application 2]

## Rating

⭐⭐⭐⭐☆ (4/5)

[One-line verdict]

## Who Should Read This
[Who would benefit from this material]
```

## Rating Guide

| Stars | Korean | English | Criteria |
|-------|--------|---------|----------|
| ⭐⭐⭐⭐⭐ | 강력 추천 | Highly Recommended | Life-changing, must-read |
| ⭐⭐⭐⭐☆ | 추천 | Recommended | Valuable, worth the time |
| ⭐⭐⭐☆☆ | 괜찮음 | Decent | Some good points, some issues |
| ⭐⭐☆☆☆ | 별로 | Not Great | Limited value |
| ⭐☆☆☆☆ | 비추천 | Not Recommended | Waste of time |

## Quick Review Template

For shorter reviews:

```markdown
## [Title] - Quick Take

**One-liner:** [What it's about in one sentence]

**Best part:** [The single most valuable thing]

**Weakest part:** [The main criticism]

**Key quote:** "[Most memorable quote]"

**Rating:** ⭐⭐⭐⭐☆

**Read if:** [Who should read this]
```

## Checklist

- [ ] Source properly cited with all metadata / 출처 정확히 인용 (메타데이터 포함)
- [ ] Core thesis captured / 핵심 논지 파악
- [ ] Key quotes selected with page numbers / 주요 인용문 페이지 번호와 함께 선택
- [ ] Personal opinion clearly stated / 개인 의견 명확히 기술
- [ ] Both agreements and critiques included / 공감점과 비판점 모두 포함
- [ ] Connections to other ideas made / 다른 아이디어와 연결
- [ ] Practical takeaways identified / 실용적 적용점 도출
- [ ] Rating provided with justification / 평점 및 근거 제공
- [ ] Target audience identified / 추천 대상 명시
- [ ] Language appropriate / 언어 적절

---

**Remember**: A good book review is not just a summary—it's a dialogue between you and the author. Share what you learned, what you questioned, and what you'll do differently.
