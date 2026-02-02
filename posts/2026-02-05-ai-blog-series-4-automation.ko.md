---
title: "도구들: AI 블로깅의 속살"
date: 2026-02-04
categories: [meta]
tags: [ai-blog-series, opencode, automation, skills, mcp, tooling]
draft: false
summary: "자동화가 실제로 어떻게 작동하는지 - 스킬, 플러그인, AI 기반 콘텐츠 제작 뒤의 기술적 배관"
series: ["Building a Blog with AI"]
cover:
  image: "/cover-series-4.png"
  alt: "톱니바퀴 아이콘이 있는 클래퍼보드"
---

## 아이디어를 넘어서: 기계장치

[이전 글](/posts/2026-02-04-ai-blog-series-3-meta/)에서 메타 재귀에 대해 이야기했다. 콘텐츠를 만드는 과정 자체가 콘텐츠가 되는 것. 그건 철학이다. 하지만 철학에는 배관이 필요하다.

이 글은 기술 인프라에 대한 것이다. "내가 일하는 동안 AI가 수확한다"는 비전이 실제로 어떻게 작동하나?

## OpenCode: 기반

모든 것은 [OpenCode](https://opencode.ai) 위에서 돌아간다. Claude용 오픈소스 CLI다. 파일을 읽고, 명령어를 실행하고, 세션 간에 컨텍스트를 유지하는 터미널 네이티브 AI 어시스턴트라고 생각하면 된다.

내가 의존하는 핵심 기능들:
- **영구 세션**: 대화가 저장되고 다시 이어갈 수 있다
- **도구 통합**: AI가 파일을 읽고 쓰고 셸 명령을 실행할 수 있다
- **MCP 지원**: Model Context Protocol 서버로 확장 가능

하지만 기본 OpenCode는 콘텐츠 씨앗이나 수확에 대해 모른다. 여기서 스킬이 등장한다.

## 스킬: AI에게 새로운 기술 가르치기

스킬은 AI의 컨텍스트에 지시사항을 주입하는 마크다운 파일이다. 단순하지만 강력하다.

```markdown
# ~/.config/opencode/skills/content-seed.md

사용자와 작업하면서 조용히 다음을 관찰해라:

**저장할 가치가 있는 프롬프트:**
- 잘 작동한 복잡하거나 새로운 요청
- AI 사용에 대한 메타 프롬프트

**캡처할 가치가 있는 아이디어:**
- "방금 깨달았는데..." 순간
- 작업 중 발견한 패턴
- 흥미로운 근거가 있는 설계 결정

**기록할 실수:**
- 버그와 수정 과정
- 시도했다가 틀린 접근법
- 디버깅 여정

씨앗 저장 위치: ~/Dev/BrainFucked/10-Blog/seeds/
```

이 스킬이 로드되면 AI는 수동적 관찰자가 된다. 질문에만 답하는 게 아니라 패턴을 알아채고 기록한다.

## 항상 켜진 스킬

처음에는 스킬을 수동으로 호출했다: `/content-seed`. 하지만 그건 수동적 패러다임을 깬다. 수확이 자동으로 일어나길 원한다.

해결책: 설정을 통한 스킬 주입.

```json
// ~/.config/opencode/oh-my-opencode.json
{
  "agents": {
    "sisyphus": {
      "skills": ["content-seed"]
    }
  }
}
```

이제 `content-seed`가 모든 세션에서 자동으로 로드된다. AI가 내가 요청하지 않아도 수확한다.

> "씨앗 아이디어 캡처와 이름 짓기는 자동화되어야 해. 나는 그냥 아이디어 내고 에이전트 작업에 피드백 주고, 너는 다 수확해"

내 세션에서 나온 저 인용문이 씨앗이 되었고, 지금 읽고 있는 이 섹션이 되었다. 메타 재귀가 작동 중이다.

## 수확 플러그인

스킬은 세션별로 로드된다. 하지만 새 작업을 시작하기 *전에* 이전 세션을 수확하려면?

플러그인이 필요하다:

```typescript
// ~/.config/opencode/plugins/harvest-on-start.ts
export default {
  name: 'harvest-on-start',
  
  onSessionStart(context) {
    // 지난 24시간 동안 수확되지 않은 세션 확인
    // 첫 메시지에 수확 지시사항 주입
  }
}
```

OpenCode가 새 세션을 시작할 때, 이 플러그인이 최근 세션의 수확을 트리거한다. 씨앗이 잊히기 전에 추출된다.

## MCP 스키마 함정

이걸 설정하는 게 순탄하지 않았다. MCP 서버 설정에서 벽에 부딪혔다.

첫 번째 시도:

```json
"mcp-image": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "mcp-image"],
  "env": { "GEMINI_API_KEY": "..." }
}
```

"Invalid input" 에러로 실패.

알고 보니 OpenCode의 MCP 스키마가 Claude Desktop과 다르다. 문서를 확인한 후:

```json
"mcp-image": {
  "type": "local",                           // "stdio"가 아님
  "command": ["npx", "-y", "mcp-image"],    // 단일 배열
  "environment": { "GEMINI_API_KEY": "..." } // "env"가 아님
}
```

세 가지 차이점. 어떤 필드가 틀렸는지 에러 메시지가 알려주지 않았다. 전형적인 설정 디버깅.

**교훈**: MCP 호스트 간 스키마 호환성을 가정하지 마라. 항상 벤더 문서를 확인해라.

## 전체 스택

이게 다 어떻게 맞물리는지:

```
[네가 일한다] → [AI가 content-seed 스킬로 관찰]
    ↓
[씨앗이 BrainFucked/10-Blog/seeds/에 기록됨]
    ↓
[harvest-on-start 플러그인] → [이전 세션 처리]
    ↓
[씨앗을 초안으로 발전]
    ↓
[Hugo가 published/에서 빌드]
    ↓
[raoulcoutard.com 블로그]
```

아름다운 점은 이것의 대부분이 보이지 않게 일어난다는 것이다. 그냥 일한다. 콘텐츠가 쌓인다.

## 이게 가능케 하는 것

이 인프라가 갖춰지면:

**제로 마찰 캡처**: 아이디어를 기록하는 데 의식적 노력이 필요 없다. 관찰되고 저장된다.

**일관된 수확**: 이전 세션이 썩지 않는다. 플러그인이 정기적 추출을 보장한다.

**관심사 분리**: 작업과 문서화가 분리된다. 나는 문제에 집중하고, 시스템이 메타 작업을 처리한다.

**재현 가능성**: 전체 설정이 설정 파일이다. 닷파일을 클론하면 워크플로우를 얻는다.

## 트레이드오프

공짜는 없다.

**오버헤드**: 스킬이 모든 프롬프트에 추가된다. 더 많은 토큰, 약간 느린 응답.

**오탐**: 가끔 AI가 보관할 가치가 없는 것도 저장한다. 수동 큐레이션은 여전히 필요하다.

**복잡성**: 단순한 "마크다운으로 쓰기" 접근법보다 움직이는 부품이 많다.

그럴 가치가 있나? 나에게는 그렇다. 아이디어를 잃지 않는 가치가 비용을 능가한다. 당신은 다를 수 있다.

## 다음 글에서

시스템이 이제 돌아간다. 씨앗이 흘러 들어온다. 하지만 씨앗은 글이 아니다. 다음 편에서는 변환 과정을 다룬다 - 원시 씨앗이 어떻게 지금 읽고 있는 다듬어진 글이 되는지.

---

*이 글은 이전 세션에서 수확한 콘텐츠 씨앗을 사용해 작성되었다. 여기 설명된 자동화가 자신에 대한 설명의 원재료를 만들었다.*

---

## 시리즈 지금까지

1. [Building a Blog with AI: The Beginning](/posts/2026-02-02-ai-blog-series-1-start/)
2. [Learning from Failure: The Value of Small Mistakes](/posts/2026-02-03-ai-blog-series-2-failures/)
3. [Content Begets Content: Discovering Meta-Recursion](/posts/2026-02-04-ai-blog-series-3-meta/)
4. [The Tools: Under the Hood of AI-Powered Blogging](/posts/2026-02-05-ai-blog-series-4-automation/)

*계속...*
