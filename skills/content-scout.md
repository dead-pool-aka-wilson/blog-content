---
name: content-scout
description: Analyze brainFucked second brain notes and recommend blog post candidates through interactive dialogue. Activate for content discovery, blog topic suggestions, or note clustering. brainFucked 노트를 분석하여 블로그 포스트 후보를 인터랙티브하게 추천. 콘텐츠 발굴, 블로그 주제 제안, 노트 클러스터링 시 활성화.
---

# Content Scout - 블로그 콘텐츠 발굴기

brainFucked 세컨드 브레인의 노트들을 분석하여 블로그 포스트로 발전시킬 수 있는 주제를 추천합니다.

## When to Activate

- User says `/content-scout` or "블로그 추천해줘"
- User wants to find blog post candidates / 블로그 글감을 찾고 싶을 때
- User asks "뭘 쓸까?" or "what should I write about?"
- User wants to review scattered notes / 흩어진 노트를 정리하고 싶을 때
- User says "brainFucked 분석해줘"

## Target Directory

**Content Location:** This repo (`blog-content/`) or linked Obsidian vault

## Scanning Workflow

### Step 1: Discover Notes (노트 탐색)

```bash
# If using blog-content repo directly:
find . -name "*.md" -type f

# If linked to full brainFucked vault:
find $BRAINFUCKED_PATH -name "*.md" -type f
```

**Target Folders (Priority Order):**

| Folder | Content Type | Priority |
|--------|-------------|----------|
| `03-Philosophy/` | 철학 글 | High |
| `04-Business/` | 비즈니스 인사이트 | High |
| `98-Draft/` | 드래프트 | High |
| `02-Technology/` | 기술 노트 | High |
| `05-Design/` | 디자인/UX | Medium |
| `06-ART/` | 예술/영화 | Medium |
| `07-Education/` | 교육/리포트 | Low |
| `97-Backlog/` | 백로그 | Low |

**Exclude:**
- `.obsidian/`
- `99-Archive/`
- `10-Blog/published/` (already published)
- `Templates/`

**Also Check MOCs:**
- `00-MOCs/*.md` - 주제별 인덱스로 클러스터 힌트 획득

### Step 2: Extract Metadata (메타데이터 추출)

For each note, extract:

```yaml
tags: [category, subtag]
created: YYYY-MM-DD
updated: YYYY-MM-DD

word_count: N
line_count: N
link_count: N
heading_count: N
has_conclusion: true/false
```

**If no frontmatter tags exist:**
- Infer category from folder path (e.g., `03-Philosophy/` → philosophy)

### Step 3: Check for Duplicates (중복 검사)

```bash
# Check published posts in this repo:
ls posts/
```

- If similar topic exists in `published/`, mark as "Already covered"
- Suggest "angle variation" if user still wants to write about it

## Completeness Evaluation (완성도 평가)

### Scoring Criteria

| Factor | Weight | Measurement |
|--------|--------|-------------|
| **Length** | 30% | Word count relative to type |
| **Structure** | 30% | Headings, sections, conclusion |
| **Connections** | 20% | Internal links, related notes |
| **Recency** | 20% | Last updated date |

### Completeness Levels

| Level | Korean | Criteria | Action Needed |
|-------|--------|----------|---------------|
| **Draft** | 초안 | <200 words, no structure | Heavy development |
| **Developing** | 발전중 | 200-500 words, some structure | Moderate work |
| **Ready** | 준비됨 | >500 words, clear structure | Light editing |
| **Published** | 발행됨 | Already in 10-Blog/published/ | Skip or update |

### Quick Assessment Questions

1. Does it have a clear thesis/main point? (명확한 주제가 있는가?)
2. Does it have supporting arguments? (뒷받침하는 논거가 있는가?)
3. Does it have a conclusion? (결론이 있는가?)
4. Is it personally meaningful? (개인적으로 의미가 있는가?)

## Clustering Method (클러스터링 방법)

### Primary: Tag-Based Grouping

```
Group notes by shared tags:
- #philosophy + #society → "사회 철학" cluster
- #business + #startup → "스타트업" cluster
- #technology + #AI → "AI 기술" cluster
```

### Secondary: Semantic Similarity

For notes without tags or cross-tag connections:
1. Analyze title and first 200 words
2. Identify key concepts and themes
3. Group by conceptual overlap

### Series Detection (시리즈 감지)

**Trigger:** 3 or more notes on same topic

**Series Recommendation Format:**
```
📚 Series Opportunity Detected!

Topic: [주제]
Related Notes:
1. [Note 1] - Ready
2. [Note 2] - Developing  
3. [Note 3] - Draft

Suggested Series Structure:
- Part 1: [Overview based on Note 1]
- Part 2: [Deep dive based on Note 2]
- Part 3: [Application based on Note 3]
```

## Effort Estimation (예상 작업량)

| Level | Korean | Time | Criteria |
|-------|--------|------|----------|
| **Quick** | 빠름 | <30분 | Ready note, minor edits only |
| **Short** | 짧음 | ~1시간 | Developing note, needs structure |
| **Medium** | 보통 | 2-3시간 | Draft note, needs significant work |
| **Long** | 김 | 반나절+ | Multiple notes to synthesize |

## Recommendation Output (추천 출력)

### Single Note Template

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 Blog Post Candidate #[N]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Title:** [노트 제목]
**Source:** [파일 경로]
**Category:** [카테고리]

**Summary:**
[2-3문장 요약]

**Why This?** (추천 이유)
- [이유 1]
- [이유 2]

**Completeness:** [Level] ████████░░ 80%
**Estimated Effort:** [Quick/Short/Medium/Long]

**Related Notes:**
- [[관련 노트 1]]
- [[관련 노트 2]]

**Suggested Next Step:**
→ /thought-dialogue - 철학적 탐구용
→ /session-to-blog - 기술 글용
→ /reading-digest - 책/기사 요약용

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[A] Accept - 이 주제로 진행
[S] Skip - 다음 추천 보기
[D] Details - 원본 노트 보기
[Q] Quit - 추천 종료
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Series Template

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📚 Series Opportunity #[N]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Theme:** [시리즈 주제]
**Notes Involved:** [N]개

**Source Notes:**
1. [노트 1] - [Completeness]
2. [노트 2] - [Completeness]
3. [노트 3] - [Completeness]

**Suggested Structure:**
- Part 1: [제목] - [설명]
- Part 2: [제목] - [설명]
- Part 3: [제목] - [설명]

**Total Estimated Effort:** [Time]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[A] Accept Series - 시리즈로 진행
[1/2/3] Single - 개별 노트만 진행
[S] Skip - 다음 추천 보기
[Q] Quit - 추천 종료
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Duplicate Warning Template

```
⚠️ Similar Content Exists!

**Your Note:** [노트 제목]
**Published Post:** [발행된 글 제목]
**Published Date:** [날짜]

**Options:**
1. Skip this topic / 이 주제 건너뛰기
2. Write from different angle / 다른 각도로 작성
3. Update existing post / 기존 글 업데이트
```

## Interactive Flow (인터랙티브 흐름)

### Session Start

```
🔍 Content Scout 시작

brainFucked 스캔 중...
- [N]개 노트 발견
- [M]개 클러스터 식별
- [K]개 시리즈 기회 감지

추천 순서: 완성도 높은 순 → 시리즈 기회 → 최근 업데이트 순

첫 번째 추천을 보시겠습니까? [Y/N]
```

### User Response Handling

| Input | Korean | Action |
|-------|--------|--------|
| `A` / `Accept` | 수락 | Show next steps, suggest related skill |
| `S` / `Skip` | 건너뛰기 | Show next recommendation |
| `D` / `Details` | 상세 | Display original note content |
| `Q` / `Quit` | 종료 | End session with summary |
| `?` / `Help` | 도움말 | Show command list |

### Session End Summary

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Content Scout 세션 요약
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Reviewed:** [N]개 추천
**Accepted:** [M]개 주제
**Skipped:** [K]개

**Accepted Topics:**
1. [주제 1] → /thought-dialogue 권장
2. [주제 2] → /session-to-blog 권장

**Next Actions:**
- 위 스킬들을 호출하여 글 작성 시작
- 또는 나중에 `/content-scout` 다시 실행

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Known High-Value Content in brainFucked

### Philosophy (03-Philosophy/)
- `진영논리에서 인간이 가진 특질이 지니는 가치란.md` - 좌우파 이데올로기와 인간 본성
- `문화대혁명과 AI.md` - 반지성주의와 AI의 비교
- `정보와 큐레이션.md` - AI 시대의 정보 수용
- `2015년 생각.md` - 기술 변화에 대한 역사적 관점

### Business (04-Business/)
- `실패할 것 같은 스타트업의 모습.md` - 스타트업 실패 신호 5가지 (Ready)
- `Inversion thinking in Product Managing.md` - 역발상 PM

### Values (03-Philosophy/Values/)
- `루이-라벨과-가치.md` - 가치 철학

## Tips (팁)

### For Better Recommendations

1. **Keep frontmatter updated** - tags와 updated 날짜 관리
2. **Use consistent tags** - 일관된 태그 사용
3. **Link related notes** - [[내부 링크]] 적극 활용
4. **Write conclusions** - 결론 섹션 추가

### Content Discovery Patterns

| Pattern | Description |
|---------|-------------|
| **Deep Dive** | 얕은 노트 여러 개 → 하나의 깊은 글 |
| **Series** | 관련 노트 3개 이상 → 시리즈 글 |
| **Update** | 오래된 Ready 노트 → 최신화 후 발행 |
| **Synthesis** | 다른 분야 노트 연결 → 융합 관점 글 |

## Checklist

- [ ] brainFucked 스캔 완료 / brainFucked scan completed
- [ ] 메타데이터 추출 완료 / Metadata extraction done
- [ ] 클러스터 식별 완료 / Clusters identified
- [ ] 중복 검사 완료 / Duplicate check done
- [ ] 추천 제시 완료 / Recommendations presented
- [ ] 사용자 선택 기록 / User selections recorded
- [ ] 다음 단계 안내 완료 / Next steps provided

---

**Remember**: This skill is READ-ONLY. It analyzes and recommends but does NOT create blog posts. After accepting a recommendation, use the appropriate skill (`/thought-dialogue`, `/session-to-blog`, `/reading-digest`, `/philosophical-interview`) to actually write the post.
