---
title: "MCP 설정 스키마: 모든 호스트가 같지 않다"
date: 2026-02-17
author: Raoul
categories: [debugging]
tags: [opencode, mcp, configuration]
draft: false
summary: "MCP 서버 설정이 Claude Desktop에서 작동했다. OpenCode에서는 검증에 실패했다. 같은 프로토콜, 다른 스키마."
lang: ko
cover:
  image: "/covers/cover-2026-02-17-mcp-config-schema-error.png"
---

```
Invalid input mcp.mcp-image
```

설정이 맞아 보였다. 봤던 MCP 예제와 일치했다. Claude Desktop에서 작동했다. 하지만 OpenCode가 시작 시 거부했다.

## 증상

다른 곳에서 작동하는 MCP 서버 설정을 복사했다:

```json
{
  "mcp": {
    "mcp-image": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-image"],
      "env": {
        "GEMINI_API_KEY": "...",
        "IMAGE_OUTPUT_DIR": "..."
      }
    }
  }
}
```

OpenCode의 검증이 즉시 실패했다. 도움이 되는 에러 메시지 없이—그냥 "Invalid input."

## 조사

첫 시도: `"type": "stdio"`를 명시적으로 필요한가? 추가했다. 여전히 실패.

돌파구는 MCP 설정이 보편적이라고 가정하는 대신 OpenCode의 실제 문서를 읽으면서 왔다. 스키마가 달랐다:

**Claude Desktop / Cline 포맷:**
```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "mcp-image"],
  "env": { ... }
}
```

**OpenCode 포맷:**
```json
{
  "type": "local",
  "command": ["npx", "-y", "mcp-image"],
  "environment": { ... }
}
```

세 가지 차이:
1. `type: "stdio"` 대신 `type: "local"`
2. `command`가 모든 args를 포함하는 배열, 별도 `args` 필드 없음
3. `env` 대신 `environment`

## 해결

```json
{
  "mcp": {
    "mcp-image": {
      "type": "local",
      "command": ["npx", "-y", "mcp-image"],
      "environment": {
        "GEMINI_API_KEY": "...",
        "IMAGE_OUTPUT_DIR": "..."
      }
    }
  }
}
```

검증 통과. 서버 시작.

## 왜 스키마가 다른가

MCP(Model Context Protocol)는 와이어 프로토콜을 정의한다—호스트와 서버가 어떻게 통신하는지. 호스트가 서버를 어떻게 설정해야 하는지는 규정하지 않는다.

각 호스트가 자체 설정 레이어를 구현한다:
- **Claude Desktop**: `command` + `args` 분리가 있는 JSON
- **OpenCode**: `command`가 배열인 JSON
- **Cline**: Claude Desktop과 유사
- **Continue**: YAML 기반 설정

MCP 명세는 메시지 포맷과 기능을 다룬다. 설정은 호스트 구현 세부사항이다.

## 더 넓은 교훈

MCP 호스트 간 설정 이식성을 가정하지 마라. 호스트 간에 서버를 이동할 때:

1. **호스트의 문서 찾기** - 일반 MCP 문서가 아닌
2. **스키마 차이 확인** - 필드 이름, 값 유형, 구조
3. **격리해서 테스트** - 작동한다고 가정하기 전에 설정 검증

호스트 간 일반적인 차이:

| 측면 | 다를 수 있음 |
|------|--------------|
| 타입 이름 | `stdio` vs `local` vs `process` |
| 명령 포맷 | 문자열 vs 배열 vs 객체 |
| Env var 필드 | `env` vs `environment` vs `envVars` |
| Args 처리 | 별도 필드 vs command 배열 내 |
| 작업 디렉토리 | 다른 필드 이름 |

## 예방 패턴

호스트별 설정을 별도 파일에 보관:

```text
~/.config/
├── claude-desktop/
│   └── mcp-servers.json     # Claude 포맷
├── opencode/
│   └── opencode.json        # OpenCode 포맷
└── mcp-servers/
    └── mcp-image/
        └── README.md        # 서버별 노트
```

새 서버를 추가할 때, 다른 호스트에서 복사하지 마라. 호스트의 문서나 예제 설정에서 시작하라.

### 로드되는 최소 설정

디버깅할 때, 가능한 가장 간단한 설정으로 시작:

**OpenCode 최소:**
```json
{
  "mcp": {
    "echo-test": {
      "type": "local",
      "command": ["echo", "hello"]
    }
  }
}
```

**Claude Desktop 최소:**
```json
{
  "mcpServers": {
    "echo-test": {
      "command": "echo",
      "args": ["hello"]
    }
  }
}
```

최소 설정이 로드되면, 문제가 되는 것을 찾을 때까지 필드를 하나씩 추가.

### 검증 체크리스트

호스트 시작 전에:

```bash
# 1. JSON 문법 확인
python3 -m json.tool < opencode.json > /dev/null && echo "JSON OK"

# 2. 필수 필드 존재 확인
jq '.mcp | to_entries[] | select(.value.type == null) | .key' opencode.json
# 모든 서버에 "type"이 있으면 아무것도 반환 안 됨

# 3. command가 배열인지 확인 (OpenCode)
jq '.mcp | to_entries[] | select(.value.command | type != "array") | .key' opencode.json
# 아무것도 반환 안 되어야 함
```

## 요약

MCP는 프로토콜이지, 설정 표준이 아니다. MCP를 말하는 각 호스트가 원하는 대로 서버를 설정할 수 있다. 와이어 포맷은 호환된다; 설정 파일은 아니다.

"Invalid input" 에러는 맞았다—내 입력은 OpenCode의 스키마에 유효하지 않았다. 다른 호스트의 스키마에는 유효했을 뿐이다.

"다른 데서 작동하는" 것이 여기서 작동하지 않으면, 망가진 서버가 아니라 스키마 불일치를 보고 있는 것이다.

설정이 한 곳에서 작동하고 다른 데서 실패하면, 올바른 호스트에 올바른 스키마를 사용하고 있는지 확인하라. 같은 프로토콜이 같은 설정을 의미하지 않는다.
