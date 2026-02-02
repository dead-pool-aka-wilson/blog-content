---
title: "API 키 유출 멈춰: .zshrc 대신 macOS 키체인을 써라"
date: 2026-02-07
author: Raoul
categories: [technical]
tags: [security, macos, keychain, api-keys, secrets-management]
draft: false
summary: "API 키는 셸 설정 파일에 있으면 안 된다. 필요할 때만 키체인에서 시크릿을 가져오는 래퍼 패턴을 소개한다."
---

지금 `~/.zshrc`에 API 키가 몇 개나 있는가?

인정한다—새 도구를 설정할 때 내 첫 본능은 항상 `export API_KEY=sk-xxx`다. 빠르고, 작동하고, 모든 튜토리얼이 이렇게 한다. 하지만 이건 현관문 매트 아래에 집 열쇠를 두는 것과 같다.

## WHY: 셸 Export의 문제

셸 설정에 `export GEMINI_API_KEY=xxx`를 추가하면, 그 키는 다음에 노출된다:

1. 터미널에서 생성된 **모든 프로세스**
2. **셸 히스토리** 파일 (종종 동기화되거나 백업됨)
3. **닷파일 저장소** (설정을 버전 관리하면)
4. 임시로라도 머신에 접근하는 **누구나**

노출 범위가 엄청나다. 키가 필요한 하나의 CLI 도구에만 주는 게 아니다. 전체 셸 환경에 영구적으로 브로드캐스트하는 것이다.

[OpenClaw](https://github.com/dead-pool-aka-wilson/openclaw) 자동화를 설정하다가 이게 클릭됐다. `export OPENCLAW_GATEWAY_TOKEN=`을 타이핑하고 즉시 생각했다: "잠깐, 이건 뭔가 잘못됐어."

내 닷파일은 GitHub에 있다. 셸 히스토리는 머신 간에 동기화된다. 실행하는 모든 프로세스가 환경 변수를 읽을 수 있다. 시크릿을 이렇게 저장하는 건 편리하지만, 게으른 보안이다.

## HOW: 키체인 래퍼 패턴

macOS에는 대부분의 개발자가 무시하는 내장 시크릿 매니저가 있다: **키체인(Keychain)**. 암호화되어 있고, 로그인 비밀번호로 보호되며, `security` 명령을 통한 CLI 인터페이스가 있다.

패턴은 간단하다: 전역으로 키를 export하는 대신, **명령이 실행될 때만** 키체인에서 시크릿을 가져오는 래퍼 스크립트를 작성한다.

먼저 키를 키체인에 저장한다:

```bash
# 키체인에 시크릿 추가
security add-generic-password \
  -a "$USER" \
  -s "gemini-api-key" \
  -w "your-actual-api-key-here"
```

그 다음, 온디맨드로 키를 가져와 export하는 래퍼 스크립트를 만든다:

```bash
#!/bin/bash
# ~/bin/openclaw - 키체인에서 가져오는 래퍼

fetch_key() {
    local keychain_name="$1"
    security find-generic-password -a "$USER" -s "$keychain_name" -w 2>/dev/null || {
        echo "Error: $keychain_name not found in Keychain" >&2
        return 1
    }
}

# 이 실행에만 시크릿 가져오기
export GEMINI_API_KEY=$(fetch_key "gemini-api-key") || exit 1
export OPENCLAW_GATEWAY_TOKEN=$(fetch_key "openclaw-gateway-token") || exit 1

# 모든 인자를 전달하며 실제 명령 실행
exec /usr/local/bin/openclaw "$@"
```

실행 권한을 주고 `~/bin`을 PATH에 추가한다:

```bash
chmod +x ~/bin/openclaw
export PATH="$HOME/bin:$PATH"  # .zshrc에 한 번만 추가
```

이제 `openclaw`를 실행하면, 래퍼가:
1. 키체인에서 시크릿을 가져옴
2. 이 프로세스에만 export
3. 실제 명령 실행
4. 프로세스가 끝나면 시크릿 사라짐

## WHAT: 보안 차이

| 접근법 | 키 노출 | 위험 수준 |
|--------|---------|-----------|
| `~/.zshrc` export | 모든 셸, 모든 프로세스, 영원히 | 높음 |
| 래퍼 스크립트 | 명령 실행 중만 | 낮음 |
| 환경 파일 (`.env`) | 다시 source할 때까지 | 중간 |

래퍼 접근법은 최소 권한 원칙을 따른다. API 키는 명령 실행 기간 동안만 메모리에 존재하고, 전체 세션 동안이 아니다.

### 여러 도구 처리

시크릿이 필요한 CLI 도구가 여러 개 있다면, 공유 헬퍼를 만든다:

```bash
# ~/.local/lib/keychain-helper.sh
fetch_keychain_secret() {
    security find-generic-password -a "$USER" -s "$1" -w 2>/dev/null
}
```

각 래퍼가 이를 source할 수 있다:

```bash
#!/bin/bash
source ~/.local/lib/keychain-helper.sh
export MY_API_KEY=$(fetch_keychain_secret "my-api-key") || exit 1
exec /path/to/tool "$@"
```

### 첫 설정

새 머신에서 설정할 때, 시크릿을 키체인에 한 번 추가해야 한다:

```bash
# 시크릿에 대한 대화형 프롬프트 (히스토리에 표시 안 됨)
read -s -p "Enter Gemini API key: " key
security add-generic-password -a "$USER" -s "gemini-api-key" -w "$key"
```

`.zshrc`에 붙여넣는 것보다 약간 더 작업이지만, 머신당 한 번 하는 작업이고, 시크릿은 안전하게 암호화되어 있다.

### 디버깅

뭔가 작동하지 않으면, 시크릿이 존재하는지 확인한다:

```bash
# 사용자의 모든 generic password 나열
security dump-keychain | grep -A 5 "gemini-api-key"

# 가져와서 확인 (주의—키가 출력됨!)
security find-generic-password -a "$USER" -s "gemini-api-key" -w
```

## 요약

모든 튜토리얼이 `export API_KEY=xxx`를 보여주는 건 설명하기 쉬워서다. 하지만 쉬운 게 안전한 건 아니다. 키체인 래퍼를 설정하는 추가 10분은 다음과 같은 상황에서 보상받는다:

- 시크릿 검사 없이 닷파일 푸시
- 데모 중 화면 공유
- 누군가 터미널 빌려줌
- 그 랜덤 npm 패키지가 환경을 읽고 있는지 궁금할 때

API 키는 유료 서비스와 민감한 데이터에 대한 액세스 토큰이다. 설정이 아니라 비밀번호처럼 취급하라.

이 패턴은 macOS에만 한정되지 않는다. Linux에는 `secret-tool`(libsecret 통해)이 있고, `pass`나 1Password CLI 같은 크로스 플랫폼 옵션도 있다. 원칙은 같다: 시크릿을 온디맨드로 가져와라, 셸에 브로드캐스트하지 마라.

현관문 매트 아래에 열쇠 두는 거 그만해라.
