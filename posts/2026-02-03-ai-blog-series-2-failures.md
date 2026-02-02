---
title: "실패에서 배우기: 작은 실수들의 가치 (2/3)"
date: 2026-02-03
categories: [meta]
tags: [ai-blog-series, debugging, bash, symlink, lessons-learned]
draft: false
summary: "블로그를 만들면서 겪은 작은 실수들과 그로부터 배운 교훈"
series: ["AI와 블로그 만들기"]
---

## 실수를 기록하는 이유

[이전 글](/posts/2026-02-02-ai-blog-series-1-start/)에서 콘텐츠 시드 시스템을 소개했다. prompts, ideas, 그리고 **misses**. 실수를 기록하는 카테고리다.

왜 실수를 기록할까?

첫째, **같은 실수를 반복하지 않기 위해**. 기록하지 않으면 잊는다. 그리고 3개월 후에 똑같이 삽질한다.

둘째, **다른 사람에게 도움이 되니까**. 내가 헤맨 길을 누군가는 피해갈 수 있다.

셋째, **실패 자체가 좋은 콘텐츠니까**. 성공 스토리보다 실패 스토리가 더 배울 게 많다. 구체적이고, 실제적이고, 공감된다.

블로그를 만들면서 겪은 세 가지 실수를 공유한다.

## 실수 1: 심링크의 함정

Hugo 블로그를 클론하고 `hugo server`를 실행했다. 잘 된다. 그런데 포스트가 안 보인다.

`content/posts/` 디렉토리를 확인했다. 파일이 없다. 이상하다. 분명 예전에 설정해뒀는데.

알고 보니 `content/posts`가 심링크였다:

```bash
ls -la content/posts
# lrwxr-xr-x  content/posts -> /Users/deok/brainFucked/10-Blog/published
```

문제가 보이는가? 현재 사용자는 `koed`인데, 심링크는 `/Users/deok/`를 가리키고 있다. 예전 맥에서 설정한 경로가 그대로 남아있었다.

**왜 이런 일이 생겼나:**

- 맥을 새로 세팅하면서 dotfiles를 복사했다
- git repo를 클론했다
- 심링크의 절대 경로는 그대로 남았다
- 에러 메시지가 없다 - 그냥 빈 디렉토리처럼 보인다

**해결:**

```bash
rm content/posts
ln -s /Users/koed/Dev/BrainFucked/10-Blog/published content/posts
```

**교훈:**

1. 가능하면 상대 경로 심링크를 써라
2. 새 머신에서 dotfiles 복사 후, 절대 경로를 확인해라
3. "아무것도 없음"이 에러일 수 있다

## 실수 2: Bash의 숨겨진 동작

테스트 스크립트를 만들었다. 첫 번째 테스트가 통과하면 카운터를 올린다. 그런데 스크립트가 첫 번째 테스트 직후에 종료된다. 에러 메시지 없이.

```bash
#!/bin/bash
set -e  # 에러 시 즉시 종료
TESTS_PASSED=0

log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++))  # 여기서 스크립트가 죽는다!
}

log_success "First test"  # 이것만 출력되고 종료
log_success "Second test" # 여기까지 안 간다
```

**무슨 일이 일어난 건가:**

`((TESTS_PASSED++))` 는 산술 표현식이다. 이 표현식의 반환값은 **증가 후 값이 아니라 증가 전 값**이다.

- `TESTS_PASSED`가 0일 때
- `((TESTS_PASSED++))` 는 0을 반환한다
- Bash에서 0은 falsy다
- `set -e` 때문에 falsy 반환값 → 스크립트 종료

첫 번째 성공에서 죽는 아이러니.

**해결:**

```bash
log_success() {
    echo "[PASS] $1"
    TESTS_PASSED=$((TESTS_PASSED + 1))  # 이건 안전하다
}
```

**교훈:**

1. `set -e`와 `((var++))` 조합은 위험하다
2. 에러 없이 조용히 종료되는 게 가장 디버깅하기 어렵다
3. Bash의 산술 연산자 반환값을 알아둬라

## 실수 3: API 키를 어디에 둘 것인가

OpenCode를 설정하면서 API 키가 필요했다. 처음에는 당연히 `~/.zshrc`에 넣으려 했다:

```bash
export GEMINI_API_KEY="sk-..."
```

그런데 뭔가 찜찜했다.

> "putting api in shell config seems not good"

맞다. 문제가 있다:

- 모든 셸 프로세스에 키가 노출된다
- 히스토리에 남을 수 있다
- dotfiles를 git으로 관리하면 실수로 커밋할 수 있다

**더 나은 방법 - macOS Keychain 래퍼:**

```bash
#!/bin/bash
# ~/bin/openclaw - 래퍼 스크립트

fetch_key() {
    local keychain_name="$1"
    security find-generic-password -a "$USER" -s "$keychain_name" -w 2>/dev/null || {
        echo "Error: $keychain_name not found in Keychain" >&2
        return 1
    }
}

export GEMINI_API_KEY=$(fetch_key "gemini-api-key") || exit 1

exec /path/to/actual/command "$@"
```

이렇게 하면:

- 키가 Keychain에 안전하게 저장된다
- 명령어 실행 시에만 키가 환경변수에 로드된다
- 명령어 종료 후 키가 사라진다
- 다른 프로세스에 노출되지 않는다

| 방식 | 키 노출 범위 |
|------|-------------|
| ~/.zshrc export | 모든 셸, 모든 프로세스 |
| 래퍼 스크립트 | 해당 명령어 실행 중에만 |

**교훈:**

1. API 키를 셸 설정 파일에 넣지 마라
2. macOS Keychain은 생각보다 쓰기 쉽다
3. 래퍼 스크립트 패턴은 범용적으로 유용하다

## 왜 이런 걸 기록하는가

이 세 가지 실수의 공통점이 있다.

1. **찾기 어렵다**: 에러 메시지가 없거나 모호하다
2. **흔하다**: 나만 겪는 게 아니다
3. **기록하지 않으면 잊는다**: 3개월 후에 또 헤맬 거다

그래서 콘텐츠 시드 시스템에 `misses/` 카테고리를 만들었다. 실수할 때마다 기록한다. 나중에 블로그 글이 되든, 그냥 나만 참고하든.

실패에서 배우려면 실패를 기록해야 한다.

## 다음 이야기

실수들을 정리하면서 흥미로운 패턴을 발견했다. 이 블로그를 만드는 과정 자체가 블로그 콘텐츠가 되고 있었다. 작업하면서 동시에 기록하고, 그 기록이 다시 콘텐츠가 된다.

마지막 글에서는 이 모든 과정에서 발견한 흥미로운 패턴에 대해 이야기하겠다.

---

*이 글에 나온 실수들은 모두 실제로 겪은 것들이고, `misses/` 시드로 먼저 기록되었다.*
