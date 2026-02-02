---
title: "존재하지 않는 Auth 파일"
date: 2026-02-16
author: Raoul
categories: [debugging]
tags: [playwright, auth, integration-bug, mismatch]
draft: false
summary: "브라우저 인증 플로우를 완료했다. 스크립트는 여전히 'Auth required'라고 했다. 셸 래퍼와 TypeScript 구현이 '인증됨'이 무엇을 의미하는지에 대해 동의하지 않았다."
---

"Auth required. Please complete the browser authentication flow."

방금 브라우저 인증 플로우를 완료했다. Google 로그인이 성공했다. NotebookLM이 열렸다. 브라우저가 상태를 저장했다. 하지만 업로드 스크립트는 여전히 인증을 요구했다.

## 증상

NotebookLM 자동화는 두 부분이 있었다:
1. 인증 상태를 확인하는 셸 래퍼 (`notebook.sh`)
2. 실제 브라우저 자동화를 하는 TypeScript 스크립트 (`upload.ts`)

인증 플로우가 성공적으로 완료됐다. 하지만 명령을 다시 실행하면 매번 같은 "Auth required" 메시지가 나왔다.

## 조사

셸 스크립트가 인증을 이렇게 확인했다:

```bash
AUTH_DIR="$HOME/.config/moltbot"
AUTH_FILE="$AUTH_DIR/notebook-lm-auth.json"

if [[ ! -f "$AUTH_FILE" ]]; then
    echo "Auth required. Please complete browser authentication."
    exit 1
fi
```

명확하다: JSON 파일을 찾고, 없으면 실패.

TypeScript 구현:

```typescript
const AUTH_DIR = join(homedir(), ".config", "moltbot", "notebook-lm-chrome");

const browser = await chromium.launchPersistentContext(AUTH_DIR, {
  headless: false,
  // ... options
});
```

TypeScript는 JSON 파일이 아닌 Chrome 프로필 디렉토리를 영속성에 사용했다. 브라우저에서 인증하면, Playwright가 쿠키, localStorage, 세션 데이터를 그 디렉토리에 저장한다. JSON 파일은 생성되지 않는다.

셸 래퍼는 `notebook-lm-auth.json`을 확인했다.
TypeScript는 `notebook-lm-chrome/`을 생성했다.

다른 산물. 다른 경로. 래퍼는 구현이 생성한 인증을 절대 보지 못할 것이었다.

## 해결

Chrome 프로필을 확인하도록 셸 스크립트 업데이트:

```bash
CHROME_PROFILE="$AUTH_DIR/notebook-lm-chrome/Default"

if [[ ! -d "$CHROME_PROFILE" ]]; then
    echo "Auth required. Please complete browser authentication."
    exit 1
fi
```

`Default/` 서브디렉토리는 프로필이 처음 사용될 때 Chromium이 생성한다. 존재하면, 인증이 완료된 것이다.

## 왜 이런 일이 일어났나

셸 래퍼가 먼저 작성됐고, 인증이 어떻게 작동할지에 대한 가정이 있었다. TypeScript 구현은 나중에 왔고 Playwright의 persistent context 기능을 사용했다—래퍼가 예상한 것과 다르게 인증을 저장하는.

개별적으로는 어느 컴포넌트도 틀리지 않았다. 그냥 둘 사이의 계약에 대해 동의하지 않았을 뿐이다.

## 더 넓은 교훈

언어 간 스크립트를 통합할 때, 실제 산물을 검증하라:

```bash
# 이전: "왜 인증이 안 작동하지?"
# 이후: "어떤 파일이 실제로 존재하지?"

ls -la ~/.config/moltbot/notebook-lm*
# 진실을 드러냄:
# notebook-lm-chrome/     <- 디렉토리 존재 (TypeScript가 이걸 생성)
# notebook-lm-auth.json   <- 없음 (셸이 이걸 예상)
```

일반적인 통합 불일치:

| 래퍼가 예상 | 구현이 생성 |
|-------------|-------------|
| JSON 설정 파일 | 디렉토리 구조 |
| 단일 인증 토큰 | 쿠키 데이터베이스 |
| 환경 변수 | 디스크의 파일 |
| 특정 파일명 | 타임스탬프 파일명 |

해결은 항상 같다: 구현이 실제로 생성하는 것을 추적하고, 래퍼가 그것을 확인하도록 업데이트.

## 예방 패턴

구현 주위에 래퍼를 작성할 때:

1. **가정하지 마라** - 구현을 실행하고 무엇이 생성되는지 봐라
2. **실제 경로 확인** - 체크를 작성하기 전에 `ls -la`
3. **계약 문서화** - "인증이 `~/.config/app/chrome-profile/`을 생성함"
4. **구현의 상수 사용** - 가능하면 구현에서 경로를 import

```bash
# 더 나음: 구현에서 유도
CHROME_PROFILE=$(node -e "console.log(require('./upload').AUTH_DIR)")
if [[ ! -d "$CHROME_PROFILE/Default" ]]; then
    echo "Auth required"
fi
```

버그는 인증 플로우에 있지 않았다. 인증 플로우는 완벽하게 작동했다. 버그는 셸이 예상한 것과 TypeScript가 제공한 것 사이의 불일치에 있었다.

"작동해야 하는" 것이 작동하지 않으면, "작동함"이 무엇을 생성하는지에 대한 가정을 확인하라.
