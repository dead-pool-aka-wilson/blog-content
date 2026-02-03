---
title: "모델을 찾을 수 없음: 설정 참조가 일치하지 않을 때"
date: 2026-02-18
author: Raoul
categories: [debugging]
tags: [opencode, oh-my-opencode, configuration]
draft: false
summary: "OpenCode가 'ProviderModelNotFoundError'로 실패했다. 모델 ID는 존재했다—내 설정이 적은 것과 다르게만."
cover:
  image: "/covers/cover-2026-02-18-model-id-mismatch.png"
---

```
ProviderModelNotFoundError
providerID: "google"
modelID: "gemini-3-flash"
suggestions: ["gemini-3-flash-preview", "antigravity-gemini-3-flash"]
```

에러가 친절하게 대안을 제안했다. 하지만 왜 프로바이더의 모델 목록에서 `gemini-3-flash`를 볼 수 있는데 내 모델 ID가 작동하지 않았나?

## 증상

내 `oh-my-opencode.json`이 커스텀 에이전트를 설정했다:

```json
{
  "agents": {
    "multimodal-looker": {
      "model": "google/gemini-3-flash"
    },
    "visual-engineering": {
      "model": "google/gemini-3-flash"
    }
  }
}
```

OpenCode가 시작을 거부했다. `gemini-3-flash`를 참조하는 모든 에이전트가 검증에 실패했다.

## 조사

에러의 `suggestions` 필드가 열쇠였다. 유효한 모델 ID를 보여줬다:
- `gemini-3-flash-preview`
- `antigravity-gemini-3-flash`

이것들은 `gemini-3-flash`와 같지 않았다. 비슷하지만 동일하지 않았다.

`opencode.json`에서 프로바이더 설정 확인:

```json
{
  "providers": {
    "google": {
      "models": [
        { "id": "antigravity-gemini-3-flash", "..." },
        { "id": "antigravity-gemini-3-pro", "..." }
      ]
    }
  }
}
```

프로바이더가 `antigravity-` 접두사로 모델을 정의했다. 내 설정은 접두사를 생략했다.

## 해결

프로바이더 설정의 정확한 ID를 사용하도록 모든 모델 참조 업데이트:

```json
{
  "agents": {
    "multimodal-looker": {
      "model": "google/antigravity-gemini-3-flash"
    },
    "visual-engineering": {
      "model": "google/antigravity-gemini-3-flash"
    }
  }
}
```

다른 에이전트 설정에 걸쳐 네 번 나타남. 모두 전체 ID가 필요했다.

## 왜 이게 중요한가

설정의 모델 ID는 프로바이더 정의와 정확히 일치해야 한다. 퍼지 매칭 없음:
- `gemini-3-flash` ≠ `antigravity-gemini-3-flash`
- 부분 일치 작동 안 함
- 대소문자 구분이 적용될 수 있음

설정 시스템은 모델 ID를 불투명 문자열로 취급한다. 문자열이 정의된 모델과 정확히 일치하지 않으면 실패한다.

## 패턴

이것은 두 설정 파일이 서로 참조할 때마다 일어난다:

```
opencode.json         oh-my-opencode.json
─────────────         ───────────────────
providers:            agents:
  google:               visual-engineering:
    models:               model: "google/???"
      - id: "xxx"                      ↑
                         정확히 일치해야 함
```

에이전트 설정이 ID로 모델을 참조한다. 그 ID들은 프로바이더 설정에 존재해야 한다. 쓸 때 검증 없음—로드할 때만.

## 모델 ID가 사는 곳

설정 계층 이해가 이 에러 방지에 도움:

```text
~/.config/opencode/
├── opencode.json           # 프로바이더와 그 모델 정의
│   └── providers.google.models[].id = "antigravity-gemini-3-flash"
│
└── oh-my-opencode.json     # ID로 모델 참조
    └── agents.visual.model = "google/???"
                                    ↑
                          opencode.json과 정확히 일치해야 함
```

모델 ID는 한 곳(`opencode.json`)에서 정의되고 다른 곳(`oh-my-opencode.json`)에서 참조된다. 참조는 정의의 정확한 문자열을 사용해야 한다.

### 일반적인 ID 드리프트 시나리오

| 시나리오 | 잘못되는 것 |
|----------|-------------|
| 문서에서 복사 | 문서는 기본 이름 보여줌, 프로바이더는 접두사 이름 사용 |
| 다른 설정에서 복사 | 다른 설정은 다른 프로바이더 설정 사용 |
| 모델 이름 변경 | 프로바이더 업데이트됨, 설정은 안 됨 |
| 오타 | `gemini-3-falsh` vs `gemini-3-flash` |

## 예방

### 1. 에러의 제안 사용
`suggestions` 필드가 유효한 대안 보여줌. 거기서 복사-붙여넣기:

```text
suggestions: ["gemini-3-flash-preview", "antigravity-gemini-3-flash"]
```

### 2. 설정 전에 사용 가능한 모델 나열
```bash
# 도구가 지원하면:
opencode models list google

# 또는 설정 직접 grep:
grep -A2 '"id":' ~/.config/opencode/opencode.json
```

### 3. 설정 파일 교차 참조
에이전트 설정 편집할 때, 프로바이더 설정 열어둬라. ID가 정확히 일치하는지 확인.

### 4. 지원되면 변수/참조 사용
일부 설정 시스템은 참조 허용:
```json
{
  "defaults": {
    "fastModel": "google/antigravity-gemini-3-flash"
  },
  "agents": {
    "visual": { "model": "${defaults.fastModel}" }
  }
}
```

모델 ID를 중앙화해서 복사-붙여넣기 에러 줄임.

### 5. 검증 스크립트 추가
```bash
#!/bin/bash
# validate-models.sh - 에이전트 설정이 유효한 모델 참조하는지 확인

DEFINED=$(grep -oP '"id":\s*"\K[^"]+' ~/.config/opencode/opencode.json)
REFERENCED=$(grep -oP '"model":\s*"[^/]+/\K[^"]+' ~/.config/opencode/oh-my-opencode.json)

for model in $REFERENCED; do
  if ! echo "$DEFINED" | grep -q "^$model$"; then
    echo "INVALID: $model"
  fi
done
```

OpenCode 재시작 전에 이걸 실행해서 불일치 조기 발견.

## 요약

에러는 정확했다: `gemini-3-flash`는 존재하지 않아서 찾지 못했다. 존재하는 모델은 `antigravity-gemini-3-flash`다. 같은 모델, 다른 ID.

설정 참조가 실패할 때:
1. 참조되는 정확한 문자열 확인
2. 존재하는 정확한 문자열 확인
3. 부분 일치가 작동한다고 가정하지 마라

설정은 문자열 매칭이다. 가깝다고 되는 게 아니다.
