---
title: "프로젝트 이름 바꾸기는 생각보다 어렵다"
date: 2026-02-15
author: Raoul
categories: [debugging]
tags: [migration, env-vars, naming, breaking-change]
draft: false
summary: "게이트웨이가 'env var 없음'으로 실패했다. 토큰은 있었다—옛 프로젝트 이름으로만."
lang: ko
cover:
  image: "/covers/cover-2026-02-15-env-var-naming-migration.png"
---

```
MissingEnvVarError: Missing env var "CLAWDBOT_GATEWAY_TOKEN"
referenced at config path: gateway.auth.token
```

분명히 그 토큰을 설정했다. 어제 이 게이트웨이를 사용했다. 뭐가 바뀌었나?

내 머신에서 바뀐 건 없다. *프로젝트*가 바뀌었다. `clawdbot`에서 `openclaw`로 이름이 바뀌었다. 설정 파일은 여전히 옛 환경 변수 이름을 참조했다.

## 증상

프로젝트가 여러 번 이름이 바뀌었다:
- `clawdbot` → `moltbot` → `openclaw`

각 이름 변경은 코드, 문서, 저장소를 업데이트했다. 하지만 환경 변수는 *사용자* 환경에 산다—셸 설정, Keychain 항목, CI 시크릿. 그것들은 커밋을 푸시해도 자동 업데이트되지 않는다.

내 설정 파일:
```json
{
  "gateway": {
    "auth": {
      "token": "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  }
}
```

내 환경:
```bash
export OPENCLAW_GATEWAY_TOKEN="actual-token"  # 새 이름
# CLAWDBOT_GATEWAY_TOKEN은 존재 안 함
```

설정이 옛 이름을 참조했다. 에러는 정확했다—그 변수는 정말로 없었다.

## 왜 이름 변경이 속임수인가

프로젝트 이름 바꾸기는 찾기-바꾸기 작업처럼 느껴진다. 아니다. 실제로 업데이트 필요한 것:

| 산물 | 위치 | 자주 놓침 |
|------|------|-----------|
| Env var 접두사 | 사용자 셸 설정 | 예 |
| 설정 파일 경로 | `~/.oldname/` 디렉토리 | 예 |
| LaunchAgent 레이블 | `~/Library/LaunchAgents/` | 예 |
| Keychain 항목 | 계정/서비스 이름 | 예 |
| 심링크 | 옛 디렉토리 가리킬 수 있음 | 예 |
| CI/CD 시크릿 | GitHub Actions 등 | 때때로 |
| 문서 | README, 위키 | 때때로 |
| 패키지 이름 | npm, pip, cargo | 보통 업데이트됨 |
| 저장소 이름 | GitHub, GitLab | 보통 업데이트됨 |

보이는 산물(레포, 패키지, 문서)은 버전 관리에 있어서 업데이트된다. 보이지 않는 산물(env vars, 로컬 설정, 시스템 서비스)은 뒤에 남는다.

## 해결

1. **새 접두사를 사용하도록 설정 업데이트:**
```json
{
  "gateway": {
    "auth": {
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

2. **Keychain 항목 이름 변경:**
```bash
# 옛 항목 삭제
security delete-generic-password -a "$USER" -s "clawdbot-gateway-token"

# 새 이름으로 추가
security add-generic-password -a "$USER" -s "openclaw-gateway-token" -w "actual-token"
```

3. 옛 이름을 참조하는 **래퍼 스크립트 업데이트**.

4. 다른 사용자를 돕기 위해 **지원 중단 경고 추가**:
```
Warning: Deprecated environment variables detected:
  CLAWDBOT_GATEWAY_TOKEN → Use OPENCLAW_GATEWAY_TOKEN
```

## 예방: 이름 변경 체크리스트

프로젝트 이름을 바꿀 때, 마이그레이션 가이드 생성:

```markdown
## oldname에서 newname으로 마이그레이션

### 환경 변수
| 옛 | 새 |
|-----|-----|
| OLDNAME_API_KEY | NEWNAME_API_KEY |
| OLDNAME_TOKEN | NEWNAME_TOKEN |

### 설정 경로
- `~/.oldname/` → `~/.newname/`
- 설정 파일: `~/.oldname/config.json` → `~/.newname/config.json`

### 시스템 서비스 (macOS)
- LaunchAgent: `com.oldname.daemon` → `ai.newname.daemon`

### 명령
실행: `newname migrate`로 로컬 설정 자동 업데이트
```

### 하위 호환성

더 부드러운 전환을 위해:

```javascript
// 옛 이름과 새 이름 둘 다 지원
const token = process.env.OPENCLAW_TOKEN || process.env.CLAWDBOT_TOKEN;

if (process.env.CLAWDBOT_TOKEN && !process.env.OPENCLAW_TOKEN) {
  console.warn('CLAWDBOT_TOKEN is deprecated. Use OPENCLAW_TOKEN');
}
```

사용자에게 업데이트할 시간을 주면서도 작동한다.

### Doctor 명령

`doctor` 또는 `diagnose` 명령이 이런 문제를 잡을 수 있다:

```bash
$ openclaw doctor

✓ Config file found
✓ API token valid
⚠ Deprecated env var CLAWDBOT_TOKEN detected
  → Rename to OPENCLAW_TOKEN

Issues found: 1 warning
```

## 교훈

코드 이름 변경은 사소했다: 찾기와 바꾸기. 생태계 이름 변경은 아니었다. 환경 변수, Keychain 항목, LaunchAgent, 로컬 설정 모두 버전 관리 밖에서 살면서, 여전히 옛 이름을 가리켰다.

프로젝트 이름 바꾸기는 저장소 밖에서 일어나는 브레이킹 체인지다. 문서화하라. 마이그레이션 도구를 제공하라. 최소 한 버전은 하위 호환성을 지원하라.

에러 메시지는 정확했다: `CLAWDBOT_GATEWAY_TOKEN`이 없었다. 하지만 진짜 버그는 프로젝트 이름 변경이 사용자 환경을 자동으로 업데이트할 거라고 가정한 것이었다.
