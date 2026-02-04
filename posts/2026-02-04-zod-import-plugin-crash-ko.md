---
title: "Zod를 직접 임포트하면 OpenCode 플러그인이 크래시되는 이유"
date: 2026-02-04
author: Raoul
categories: [debugging, lessons-learned]
tags: [opencode, zod, plugin, typescript, debugging]
draft: false
summary: "OpenCode 플러그인에서 툴 스키마를 정의할 때 'undefined is not an object' 에러를 발생시킨 미묘한 버전 불일치"
lang: ko
cover:
  image: "/covers/2026-02-04-zod-import-plugin-crash.png"
---

## 무슨 일이 있었나

툴 정의가 있는 커스텀 OpenCode 플러그인을 만들고 있었습니다. TypeScript 타입을 따라서 Zod를 직접 임포트해 스키마를 정의했습니다:

```typescript
import { z } from "zod";

export default {
  name: "my-plugin",
  tools: [
    {
      name: "my-tool",
      description: "뭔가 유용한 작업",
      args: {
        input: z.string().describe("입력값"),
      },
      execute: async ({ input }) => {
        // ...
      },
    },
  ],
};
```

시작할 때 OpenCode가 크래시했습니다:

```
TypeError: undefined is not an object (evaluating 'schema._zod.def')
    at toJSONSchema (node_modules/.bun/zod@4.1.8/node_modules/zod/v4/core/to-json-schema.js:211:57)
    at resolveTools (src/session/prompt.ts:704:62)
```

## 근본 원인

에러는 `zod@4.1.8`을 가리켰습니다—OpenCode의 내부 Zod 버전. 하지만 제 플러그인은 *제* node_modules에서 Zod를 임포트하고 있었고, 이건 다른 버전(또는 완전히 다른 Zod 인스턴스)이었습니다.

문제: **Zod 스키마는 크로스 인스턴스 호환이 안 됨**.

OpenCode가 제 스키마를 JSON Schema로 직렬화하려 할 때, 자체 Zod 인스턴스에만 존재하는 `_zod.def` 같은 내부 속성을 기대했습니다. 제 플러그인의 Zod 스키마는 다른 객체 구조였습니다.

이것은 `instanceof` 체크나 내부 속성을 사용하는 라이브러리의 흔한 문제입니다. 다른 버전의 같은 라이브러리 두 복사본—또는 같은 버전이라도 두 번 설치된 경우—은 호환되지 않는 객체를 만듭니다.

## 해결책

Zod 스키마 대신 JSON Schema 형식을 직접 사용하세요:

```typescript
export default {
  name: "my-plugin",
  tools: [
    {
      name: "my-tool",
      description: "뭔가 유용한 작업",
      parameters: {
        type: "object",
        properties: {
          input: {
            type: "string",
            description: "입력값",
          },
        },
        required: ["input"],
      },
      execute: async ({ input }) => {
        // ...
      },
    },
  ],
};
```

핵심 변경: `args` (Zod) → `parameters` (JSON Schema)

TypeScript 타입은 Zod와 함께 `args`를 제안하지만, 런타임은 순수 JSON Schema와 함께 `parameters`를 받아들입니다. 이것은 크로스 인스턴스 호환성 문제를 완전히 피합니다.

## 대안: OpenCode의 Zod 사용

Zod의 편리함이 필요하다면, OpenCode의 플러그인 유틸리티에서 임포트하세요:

```typescript
import { tool } from "@opencode-ai/plugin/tool";

// OpenCode와 같은 Zod 인스턴스를 사용
const myTool = tool({
  name: "my-tool",
  description: "뭔가 유용한 작업",
  schema: z => z.object({
    input: z.string().describe("입력값"),
  }),
  execute: async ({ input }) => {
    // ...
  },
});
```

콜백 패턴은 올바른 Zod 인스턴스를 사용하도록 보장합니다.

## 배운 점

1. **패키지 경계를 넘어선 라이브러리 호환성을 가정하지 마세요** - 각 `node_modules`는 자체 복사본을 가질 수 있음
2. **내부 속성 (`_zod`, `_internal`)은 위험 신호** - 버전마다 다를 수 있는 구현 세부사항을 나타냄
3. **JSON Schema가 보편적인 교환 형식** - 의심스러우면 최소 공통분모를 사용
4. **TypeScript 타입이 런타임 동작을 보장하지 않음** - 타입은 `args`라고 했지만 런타임은 `parameters`를 원했음

## 예방법

프레임워크 플러그인을 만들 때:

- 스키마를 정의하는 표준 방법을 문서에서 확인
- 직접 라이브러리 임포트보다 프레임워크 제공 유틸리티 선호
- 통합하기 전에 플러그인을 격리된 환경에서 테스트
- 공유 라이브러리를 사용해야 한다면 호스트와 버전 일치 확인

이것 때문에 2시간 디버깅했습니다. 에러 메시지는 암호 같았고, 실제 문제—패키지 경계를 넘나드는 호환되지 않는 Zod 인스턴스—가 아닌 Zod 내부를 가리켰습니다.
