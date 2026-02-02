---
title: "씨앗에서 글로: 변환 과정"
date: 2026-02-05
categories: [meta]
tags: [ai-blog-series, content-creation, writing-process, workflow]
draft: false
summary: "원시 콘텐츠 씨앗이 완성된 블로그 글이 되는 방법 - 큐레이션, 발전, 변환 과정"
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-5.png"
  alt: "새싹과 꽃이 있는 클래퍼보드"
---

## 수확은 시작일 뿐

[이전 글](/posts/2026-02-05-ai-blog-series-4-automation/)에서 기술 인프라를 설명했다 - 스킬, 플러그인, MCP 서버. 하지만 인프라는 절반의 이야기일 뿐이다.

씨앗이 쌓인다. 좋다. 그래서?

씨앗은 글이 아니다. 원재료다 - 반쯤 형성된 생각, 코드 스니펫, 그 순간에 포착된 디버깅 스토리. 변환이 필요하다.

## 씨앗 리뷰

며칠마다 쌓인 것들을 본다:

```
seeds/
├── ideas/
│   ├── 20260201-meta-content-creation.md      ← 높은 잠재력
│   ├── 20260201-keychain-wrapper-secrets.md   ← 높은 잠재력
│   ├── 20260202-lenis-scrollsnap-conflict.md  ← 중간 잠재력
│   └── ...
├── prompts/
│   └── 20260201-content-seeds-system.md       ← 이미 사용됨
└── misses/
    ├── 20260201-symlink-wrong-path.md         ← 이미 사용됨
    ├── 20260201-bash-arithmetic-gotcha.md     ← 이미 사용됨
    └── ...
```

각 씨앗에는 `content_potential` 등급이 있다: 높음, 중간, 낮음. 캡처할 때 부여되지만, 종종 틀린다. 새벽 2시에 훌륭해 보였던 아이디어가 낮에 보면 평범해 보인다.

### 필터링 기준

모든 씨앗이 글이 되진 않는다. 좋은 후보는:

**보편성**: 내 특정 상황을 넘어 적용되나? 잘못된 사용자 경로를 가리키는 심링크는 보편적이다. 내 특정 코드베이스의 버그는 아니다.

**가르칠 수 있음**: 명확하게 설명할 수 있나? 어떤 아이디어는 심오하게 느껴지지만 표현을 거부한다. 발전할 시간이 더 필요하다.

**스토리**: 서사가 있나? 디버깅 여정은 잘 작동한다. "X를 설정했다"는 그렇지 않다.

**신선함**: 이미 많이 다뤄졌나? 또 다른 "Hugo 설정 방법" 글은 누구에게도 도움이 안 된다.

## 발전 과정

씨앗이 선택되면 초안으로 옮긴다:

```bash
mv seeds/ideas/20260201-keychain-wrapper-secrets.md drafts/
```

그 다음 확장이 시작된다. 씨앗은 보통 다음을 가진다:
- 한 줄 아이디어
- 약간의 컨텍스트
- 아마도 코드 스니펫이나 인용문

글에는 필요하다:
- 오프닝 훅
- 구조화된 섹션
- 구체적인 예시
- 결론

### AI 보조 확장

여기서 AI 협업이 빛난다. `/thought-dialogue` 스킬을 사용한다:

> "키체인 래퍼 아이디어를 발전시키자. 왜 누군가가 관심을 가질지부터 시작해."

AI가 질문한다:
- "이게 ~/.zshrc가 해결 못하는 어떤 문제를 해결하나?"
- "대상 독자가 누구야 - 보안에 민감한 개발자? 초보자?"
- "전체 래퍼 스크립트를 포함할 건지 개념만?"

대화를 통해 씨앗이 확장된다. 혼자 글을 쓰는 게 아니라 - 산문을 생성하는 대화를 나누고 있다.

### 스킬 스택

다른 단계에 다른 스킬:

| 스킬 | 언제 사용 |
|-------|-------------|
| `/content-seed` | 수동적 캡처 (항상 켜짐) |
| `/thought-dialogue` | 원시 아이디어 발전 |
| `/session-to-blog` | 작업 세션을 직접 변환 |
| `/reading-digest` | 외부 소스 요약 |

적절한 순간에 적절한 스킬을.

## 초안에서 발행까지

초안은 다음이 될 때까지 발행되지 않는다:

1. **구조 확인**: 글이 흐르는가? 헤더가 논리적 순서인가?
2. **예시 확인**: 모든 주장이 코드나 구체적 예시로 뒷받침되는가?
3. **프론트매터 완성**: Hugo는 `title`, `date`, `categories`, `tags`, `summary`, `series`가 필요하다
4. **번역 완료**: 이 블로그는 한국어와 영어 버전 둘 다

### 번역 문제

이 블로그는 한국어와 영어를 지원한다. 모든 글에 둘 다 필요하다.

영어로 먼저 쓴다 (나에게는 기술 주제에 대해 영어로 생각하는 게 더 쉽다). 그 다음 AI가 번역을 돕는데, 직역이 아니다:

> "이 글을 한국어로 번역해. 대화체 톤 유지해. 격식체 존댓말 쓰지 마 - 해체 스타일로."

번역은 창작 작업이지 기계적 작업이 아니다. 영어에서 작동하는 표현이 한국어에서는 완전히 재구성이 필요할 수 있다.

## 발행 플로우

두 버전이 준비되면:

```
drafts/keychain-wrapper.md
    ↓ (published/로 이동)
published/2026-02-10-keychain-wrapper.md
published/2026-02-10-keychain-wrapper.ko.md
    ↓ (Hugo에 심링크됨)
content/posts/2026-02-10-keychain-wrapper.md
    ↓ (hugo build)
public/posts/2026-02-10-keychain-wrapper/index.html
```

`content/posts/`에서 `BrainFucked/10-Blog/published/`로의 심링크는 Obsidian과 Hugo가 같은 파일을 본다는 뜻이다. 한 곳에서 편집하고, 다른 곳에서 배포한다.

## 숫자들

지금까지 세션에서:

| 지표 | 개수 |
|--------|-------|
| 캡처된 씨앗 | 21 |
| 글에 사용됨 | 7 |
| 전환율 | 33% |
| 진행 중 초안 | 3 |

모든 게 글이 되진 않는다. 괜찮다. 전환되지 않은 씨앗도 낭비가 아니다 - 미래 사고에 영향을 주고, 다른 글에 스니펫을 제공하거나, 적절한 순간까지 그냥 있다.

## 이게 작동하는 이유

핵심 통찰: **모든 단계에서 장벽을 낮춰라**.

- **캡처 장벽**: 거의 제로. AI가 자동으로 수확한다.
- **리뷰 장벽**: 낮음. 제목만 훑고 잠재력 등급 확인.
- **발전 장벽**: 낮춰짐. AI가 대화를 통해 확장을 돕는다.
- **번역 장벽**: 낮춰짐. AI가 첫 번째 패스를 처리한다.

각 단계에서 마찰이 제거된다. 완전히 제거되진 않는다 - 창작 작업은 여전히 노력이 필요하다 - 하지만 콘텐츠가 실제로 흐를 만큼 충분히 줄어든다.

## 메타 예시

지금 읽고 있는 이 글도 같은 과정을 거쳤다:

1. **씨앗**: drafts/ → published/ 워크플로우에 대한 메모
2. **발전**: `/thought-dialogue`로 변환에 대해 확장
3. **초안**: harvest 워크트리에서 작성
4. **번역**: 대기 중
5. **발행**: 지금

시스템이 자기 자신을 문서화한다.

---

*이 글은 그것을 쓰는 장벽이 실제로 쓸 만큼 충분히 낮았기 때문에 존재한다.*

---

## 시리즈 지금까지

1. [Building a Blog with AI: The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Content Begets Content: Discovering Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [The Tools: Under the Hood of AI-Powered Blogging](/posts/2026-02-05-ai-blog-series-4-automation/)
5. [From Seeds to Posts: The Transformation](/posts/2026-02-06-ai-blog-series-5-transformation/)

*계속...*
