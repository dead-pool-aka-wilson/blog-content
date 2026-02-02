---
title: "좋은 시스템 설계 프롬프트의 해부학"
date: 2026-02-20
author: Raoul
categories: [ai-prompting]
tags: [system-design, content-workflow, meta]
draft: false
summary: "한 문장이 전체 콘텐츠 캡처 시스템을 시작했다. 모호하게 들리는 프롬프트가 놀랍도록 효과적이었던 이유 분석."
---

> "in dev folder of koed I want to gather prompts I wrote, Idea I had, misses we made and write blog contents with those. for example let's say i am creating blog with you right now, and I want this to be an content at the same time."

이 프롬프트는 특별해 보이지 않는다. 구조화된 포맷 없음. 명시적 요구사항 없음. 문법적으로 비형식적. 하지만 단일 명확화 질문 없이 완전한 콘텐츠 캡처 시스템—디렉토리 구조, 템플릿, 자동화, 문서—을 생성했다.

뭐가 작동하게 만들었나?

## 프롬프트

다시 인용해보자, 단순함이 포인트니까:

> "in dev folder of koed I want to gather prompts I wrote, Idea I had, misses we made and write blog contents with those. for example let's say i am creating blog with you right now, and I want this to be an content at the same time."

45 단어. 불릿 포인트 없음. 명세 문서 없음. 그런데 필요한 모든 것을 담았다.

## 작동하게 만든 것

### 1. 명확한 카테고리

명시적으로 명명된 세 가지 콘텐츠 유형:
- **프롬프트(Prompts)** - AI에 쓴 지시
- **아이디어(Ideas)** - 통찰과 개념
- **실수(Misses)** - 실수와 실패

이 카테고리들은 구별되고 겹치지 않는다. AI가 즉시 세 폴더가 있는 디렉토리 구조를 제안할 수 있었다.

### 2. 구체적인 위치

"in dev folder of koed" - 실제 파일시스템 경로. "편리한 어딘가"나 "좋은 곳"이 아닌. 구체적인 시작점.

이 그라운딩이 물건이 어디에 *갈 수 있는지*에 대한 추상적 논의를 방지했다. 물건이 어디에 *갈 것인지*를 말했다.

### 3. 메타 인식

"let's say i am creating blog with you right now, and I want this to be an content at the same time"

캐주얼한 문구에 묻힌 핵심 통찰이다. 대화 자체가 캡처되어야 한다. 시스템을 설계하는 행위도 시스템의 콘텐츠다.

이 메타 지시가:
- 즉각적인 사용 사례를 명확히 함
- 작업할 실제 예제 제공
- 제약 생성 (시스템이 이 대화를 처리해야 함)

### 4. 열린 설계 공간

프롬프트가 *무엇*을 명시했지만 *어떻게*는 아니었다. 파일 포맷, 네이밍 컨벤션, 자동화에 대한 요구사항 없음. 이것이 AI가 모범 사례에 기반해 솔루션을 제안할 여지를 남겼다.

과도하게 구체적인 프롬프트("정확히 이 필드들로 YAML frontmatter 사용...")는 설계를 제약했을 것이다. 열어두면 발견이 가능했다.

## 응답 품질

AI의 응답이 이해를 보여줬다:

1. **구현 전 제안** - 디렉토리 구조를 보여주고 확인 요청
2. **기존 컨텍스트 기반 구축** - 기존 블로그 스킬 발견하고 통합
3. **실제 예제 생성** - 템플릿만이 아닌 실제 첫 항목
4. **구축하면서 문서화** - 시스템 README가 시스템 구축의 일부로 생성

명확화 질문 불필요. 프롬프트가 행동하기에 충분히 구체적이고, 좋은 설계를 허용하기에 충분히 열려있었다.

## 패턴

효과적인 시스템 설계 프롬프트는 구체성과 개방성의 균형:

| 요소 | 이 프롬프트 | 왜 작동하는가 |
|------|-------------|---------------|
| 무엇을 만들까 | "gather prompts, ideas, misses" | 명확한 범위 |
| 어디에 만들까 | "dev folder of koed" | 그라운드된 위치 |
| 왜 만들까 | "write blog contents" | 명확한 목적 |
| 예제 사용 사례 | "creating blog with you right now" | 즉각적 검증 |
| 어떻게 만들까 | (명시 안 됨) | 설계 유연성 |

프롬프트가 "무엇, 어디, 왜"에 답하고 "어떻게"는 구현자에게 남겼다.

## 이 패턴이 작동할 때

모호하지만-그라운드된 프롬프트 사용:
- 구현자(AI든 인간이든)의 판단을 신뢰할 때
- 명세 준수가 아닌 창의적 솔루션을 원할 때
- 도메인에 끌어올 확립된 패턴이 있을 때
- 결과에 대해 반복할 수 있을 때

상세 명세 사용:
- 정확한 호환성 요구사항이 있을 때
- 여러 구현자가 조율 필요할 때
- 솔루션이 기존 시스템과 정확히 일치해야 할 때
- 반복이 비쌀 때

## 안티 패턴

반대 극단—과도하게 명시된 프롬프트—종종 더 나쁜 결과:

```
다음 요구사항으로 콘텐츠 캡처 시스템 생성:
- 디렉토리 구조: seeds/prompts, seeds/ideas, seeds/misses
- 파일 포맷: YAML frontmatter가 있는 Markdown
- Frontmatter 필드: date (ISO 8601), tags (array), content_potential (enum: low/medium/high)
- 네이밍 컨벤션: YYYYMMDD-slug.md
- 각 유형에 대한 템플릿 포함
- 셸 명령 생성: seed-prompt, seed-idea, seed-miss
- ...
```

이 프롬프트는 이미 올바른 설계를 안다고 가정한다. 올바른 설계를 알았으면 구축하는 데 도움이 필요 없었을 것이다.

## 요약

이 블로그의 콘텐츠 시스템을 시작한 프롬프트는 비형식적이고, 짧고, 구조가 없었다. 작동한 이유:

1. 카테고리가 명확 (prompts, ideas, misses)
2. 위치가 구체적 (dev folder)
3. 목적이 진술됨 (blog content)
4. 실제 예제가 내장됨 (this conversation)
5. 구현은 열려있음

시스템 설계 도움을 요청할 때, 문제를 이해하기에 충분한 맥락을 제공하고 비켜라. 최고의 프롬프트는 솔루션을 지시하기보다 초대한다.

내 것은 45 단어였다. 여전히 사용 중인 시스템을 구축했다.
