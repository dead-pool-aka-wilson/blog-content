---
title: "실패에서 배우기: 작은 실수들의 가치"
date: 2026-02-03
author: Raoul
categories: [meta]
tags: [ai-blog-series, debugging, bash, symlink, lessons-learned]
draft: false
summary: "블로그를 만들면서 겪은 세 가지 구체적인 실패와 그로부터 배운 교훈들을 공유합니다."
series: ["AI와 블로그 만들기"]
cover:
  image: "/cover-series-2.png"
  alt: "Film clapperboard with X marks"
---

혹시 3시간 동안 헤매던 버그가 알고 보니 예전에 내가 짠 코드 한 줄 때문이었던 적 있으신가요? 저는 생각보다 자주 그런 일을 겪습니다.

[이전 글](/ko/posts/2026-02-02-ai-blog-series-1-start/)에서 콘텐츠 시드 시스템을 소개했었죠. prompts, ideas, 그리고 **misses**. 여기서 misses는 바로 실수를 기록하는 카테고리입니다.

왜 굳이 내 실수를 기록할까요? 

첫째, 똑같은 실수를 반복하지 않기 위해서입니다. 기록하지 않으면 잊어버리고, 3개월 뒤에 똑같이 삽질하게 되거든요. 둘째, 다른 사람에게 도움이 되기 때문입니다. 제가 헤맨 길을 누군가는 피해갈 수 있다면 그것만으로 가치가 있죠. 마지막으로, 실패는 그 자체로 훌륭한 콘텐츠입니다. 성공담은 가공되어 멀게 느껴질 때가 많지만, 실패담은 구체적이고 생생해서 더 배울 게 많거든요.

이 블로그를 만들면서 겪은 세 가지 '실수'를 공유합니다.

## 1. 심링크의 함정

블로그 저장소를 새로 클론하고 `hugo server`를 실행했습니다. 서버는 잘 뜨는데, 포스트가 하나도 안 보입니다.

`content/posts/` 디렉토리를 확인해 보니 텅 비어있었습니다. 분명히 예전에 설정을 다 해뒀는데 이상했죠. 파일 목록을 자세히 살펴보고 나서야 범인을 찾았습니다.

```bash
ls -la content/posts
# lrwxr-xr-x  content/posts -> /Users/olduser/brainFucked/10-Blog/published
```

범인은 심링크였습니다. 현재 사용자는 `newuser`인데, 심링크는 여전히 `/Users/olduser/`를 가리키고 있었던 거죠. 예전 맥에서 설정한 경로가 dotfiles를 타고 그대로 넘어온 겁니다.

**왜 그랬을까요?**
Git 저장소가 같으니 환경도 당연히 똑같을 거라고 생각했습니다. **절대 경로는 특정 머신의 '영혼'에 묶여 있다**는 사실을 간과한 거죠.

**해결 방법:**
심링크를 현재 머신에 맞는 경로로 업데이트했습니다.
```bash
rm content/posts
ln -s /Users/newuser/Dev/BrainFucked/10-Blog/published content/posts
```

**교훈:**
심링크를 쓸 때는 가능하면 상대 경로를 사용하세요. 그리고 무언가 있어야 할 곳에 '아무것도 없을' 때는 코드보다 먼저 배관(경로)을 확인하는 게 빠릅니다.

## 2. Bash의 조용한 죽음

간단한 테스트 스크립트를 하나 짰습니다. 테스트가 통과할 때마다 카운터를 올리게 했죠. 그런데 스크립트가 첫 번째 테스트 직후에 그냥 종료되어 버렸습니다. 에러 메시지도 없이요.

```bash
#!/bin/bash
set -e  # 에러 시 즉시 종료
TESTS_PASSED=0

log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++))  # 여기서 스크립트가 조용히 죽습니다!
}

log_success "First test"
log_success "Second test"
```

**무슨 일이 일어난 걸까요?**
문제는 `((TESTS_PASSED++))` 산술 표현식에 있었습니다. Bash에서 이 표현식은 '증가 전' 값을 반환합니다.

1. `TESTS_PASSED`가 0일 때,
2. `((TESTS_PASSED++))`는 0을 반환합니다.
3. Bash에서 0은 '거짓(falsy)' 또는 에러 상태를 의미합니다.
4. `set -e` 옵션 때문에 Bash는 이를 에러로 판단하고 스크립트를 즉시 종료시킨 겁니다.

첫 번째 성공을 기록하자마자 스크립트가 죽다니, 참 아이러니하죠.

**해결 방법:**
반환값에 의존하지 않는 안전한 방식으로 카운터를 올렸습니다.
```bash
log_success() {
    echo "[PASS] $1"
    TESTS_PASSED=$((TESTS_PASSED + 1))
}
```

**교훈:**
`set -e`는 버그를 잡는 데 유용하지만, Bash의 미묘한 문법과 만나면 지뢰가 됩니다. 특히 산술 연산의 반환값을 다룰 때는 조심하세요. Bash의 논리는 우리가 생각하는 것보다 훨씬 오래되고 독특합니다.

## 3. API 키는 어디에 두어야 할까

OpenCode를 설정하면서 Gemini API 키를 저장해야 했습니다. 처음엔 무심코 `~/.zshrc`에 넣으려고 했죠.

```bash
export GEMINI_API_KEY="sk-..."
```

그런데 왠지 찜찜했습니다. 셸 설정 파일에 키를 export 하는 건 현관 열쇠를 매트 밑에 두는 것과 비슷하거든요. 모든 셸 프로세스에 키가 노출되고, 히스토리에 남을 수도 있으며, 실수로 dotfiles에 포함해 커밋할 위험도 있습니다.

**대안: macOS Keychain 래퍼**
전역 변수로 노출하는 대신, 필요할 때만 Keychain에서 키를 가져오는 래퍼 스크립트를 만들었습니다.

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

**왜 이 방식이 더 좋을까요?**
키는 시스템 Keychain에 안전하게 저장됩니다. 오직 해당 명령어가 실행될 때만 메모리에 로드되고, 명령어가 종료되면 환경 변수에서도 사라집니다. 노출 범위(blast radius)를 최소한으로 줄인 거죠.

**교훈:**
기본적인 방식이 항상 최선은 아닙니다. 약간의 수고를 들여 '배관'을 잘 짜두면 보안 위험을 획기적으로 낮출 수 있습니다.

## 실패의 영수증을 챙기세요

이 세 가지 실수의 공통점은 찾기는 어렵지만 해결은 간단했다는 점입니다. 

만약 제가 `misses/` 카테고리에 기록해두지 않았다면, 아마 6개월 뒤에 똑같은 실수를 반복했을 겁니다. 실패를 콘텐츠로 만들면서 저는 **왜** 이런 일이 생겼는지 돌아보게 되었고, 그제야 진짜 배움이 일어났습니다.

기록하지 않은 실패는 그저 시간 낭비일 뿐이지만, 기록된 실패는 성장의 자산이 됩니다.

## 다음 이야기

실수들을 정리하다 보니 흥미로운 패턴을 발견했습니다. 이 블로그를 만드는 과정 자체가 블로그 콘텐츠가 되고 있었거든요. 시리즈의 마지막 글에서는 이 '메타 재귀'의 발견과 그것이 제 글쓰기에 어떤 변화를 주었는지 이야기해보겠습니다.

---

## 시리즈

1. [AI와 함께 블로그를 만들다: 시작](/ko/posts/2026-02-02-ai-blog-series-1-start/)
2. [실패에서 배우기: 작은 실수들의 가치](/ko/posts/2026-02-03-ai-blog-series-2-failures/)
3. [콘텐츠가 콘텐츠를 낳다: 메타 재귀의 발견](/ko/posts/2026-02-04-ai-blog-series-3-meta/)
