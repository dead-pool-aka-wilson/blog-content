---
title: "내 스크립트를 죽인 Bash 증가 연산자"
date: 2026-02-14
author: Raoul
categories: [debugging]
tags: [bash, gotcha, set-e, arithmetic]
draft: false
summary: "테스트 스크립트가 첫 번째 통과 테스트 후 조용히 죽었다. 범인: set -e와 결합된 ((var++))."
lang: ko
cover:
  image: "/covers/cover-2026-02-14-bash-increment-gotcha.png"
---

스크립트가 완벽하게 작동했다—정확히 하나의 테스트에 대해.

실패를 잡기 위해 `set -e`로 테스트 러너를 작성했다. 각 통과 테스트가 카운터를 증가시켰다. 첫 번째 테스트가 통과하고, 성공을 로깅하고, 그 다음... 아무것도. 스크립트가 조용히 종료됐다. 에러 메시지 없음. 뭐가 잘못됐는지 표시 없음.

## 증상

```bash
#!/bin/bash
set -e

TESTS_PASSED=0

log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++))  # 스크립트가 여기서 죽음
}

# 테스트
log_success "Test 1"  # "[PASS] Test 1" 출력 후 종료
log_success "Test 2"  # 도달 안 됨
log_success "Test 3"  # 도달 안 됨

echo "Passed: $TESTS_PASSED"  # 도달 안 됨
```

이 스크립트를 실행하라. `[PASS] Test 1`이 보이고 프롬프트로 돌아간다. "Test 2" 없고, 최종 카운트 없음. 스크립트가 첫 번째 증가 후 종료된다.

## 조사

디버깅을 위해 `set -x`를 추가하니 `((TESTS_PASSED++))` 직후에 스크립트가 종료되는 게 보였다. 하지만 왜? 증가는 해가 없어야 하는데.

통찰은 bash 산술 평가가 어떻게 작동하는지 이해하면서 왔다:

```bash
TESTS_PASSED=0
((TESTS_PASSED++))
echo $?  # "1" 출력 - 실패!
```

`(( ))` 구문은 표현식의 값에 기반한 종료 코드를 반환한다:
- 0이 아닌 결과 → 종료 코드 0 (성공)
- 0 결과 → 종료 코드 1 (실패)

`((TESTS_PASSED++))`는 후위 증가다. 증가 *전*의 값을 반환한다. `TESTS_PASSED`가 0일 때, 표현식은 0으로 평가되고, bash는 이를 false로 취급하고, 종료 코드 1을 의미한다.

`set -e`와 함께, 0이 아닌 것을 반환하는 모든 명령이 스크립트를 종료시킨다. 증가는 "성공"하지만 (변수는 업데이트됨), 반환 값이 falsy라서, `set -e`가 스크립트를 죽인다.

## 해결

증가 연산자 대신 명시적 산술 할당 사용:

```bash
log_success() {
    echo "[PASS] $1"
    TESTS_PASSED=$((TESTS_PASSED + 1))  # 항상 0 반환
}
```

또는 종료 코드 억제:

```bash
log_success() {
    echo "[PASS] $1"
    ((TESTS_PASSED++)) || true  # 성공 강제
}
```

또는 전위 증가 사용 (0에서 시작하지 않을 때만 작동):

```bash
TESTS_PASSED=-1  # 해키함, 하지 마라
((++TESTS_PASSED))  # 전위 증가는 새 값 반환
```

가장 깔끔한 해결은 첫 번째: `VAR=$((VAR + 1))`은 종료 코드 문제가 없다.

## 왜 이게 악랄한가

이 버그는 잡기 어렵다 왜냐면:

1. **에러 메시지 없음** - 스크립트가 그냥 멈춤
2. **성공 시 실패** - 첫 번째 *통과* 테스트가 죽임
3. **코드가 맞아 보임** - `((var++))`은 관용적 bash
4. **타이밍 의존적** - 0에서 시작할 때만 실패

카운터가 1에서 시작하거나, 감소 중이거나, 이미 한 번 증가했으면, 이 버그를 절대 보지 못한다. 0에서의 첫 번째 증가에서만 트리거된다.

## 더 넓은 교훈

`set -e`는 유용하지만 위험하다. 사소한 bash 특이점을 스크립트 종료 지뢰로 바꾼다. 다른 `set -e` 함정들:

```bash
# 이것들은 모두 놀라운 종료 코드를 가진다:
local var=$(false)      # Local은 명령이 실패해도 성공
var=$(false) || true    # || true에 절대 도달 안 됨
grep "pattern" file     # 패턴 못 찾으면 종료 1 (에러 아님)
diff file1 file2        # 파일이 다르면 종료 1 (에러 아님)
```

가장 안전한 접근: 중요한 스크립트에 `set -e` 사용하되, 까다로운 종료 코드가 있는 명령은 명시적으로 처리.

```bash
set -e

# 잠재적 "실패" 명시적 처리
count=$(grep -c "pattern" file) || count=0
TESTS_PASSED=$((TESTS_PASSED + 1))  # 안전한 증가
```

## 요약

자체 성공 카운터에 의해 죽고 있는 테스트 프레임워크를 디버깅하느라 30분을 썼다. 해결은 `((var++))`를 `var=$((var + 1))`로 바꾸는 것이었다.

Bash는 이런 날카로운 모서리로 가득하다. 증가 연산자의 반환 값 동작은 문서화되어 있지만, 직관적이지 않고, `set -e`와 나쁘게 상호작용한다.

스크립트가 정확히 해야 할 일을 한 후 조용히 죽으면, 종료 코드 함정을 의심하라. 그리고 `set -e` 스크립트에서 `((var++))`를 절대 믿지 마라.
