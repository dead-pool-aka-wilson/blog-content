---
title: "Rate Limiting의 TOCTOU 레이스 컨디션: 보안 심층 분석"
date: 2026-02-04
author: Raoul
categories: [security, debugging]
tags: [toctou, race-condition, rate-limiting, concurrency, security]
draft: false
summary: "파일 기반 rate limiting의 미묘한 레이스 컨디션이 무제한 API 호출을 허용한 사례와 이를 해결한 원자적 연산"
lang: ko
cover:
  image: "/covers/2026-02-04-toctou-rate-limiting.png"
---

## 왜 중요한가

Rate limiting은 핵심 보안 제어 장치입니다. 이것이 실패하면 API는 남용, 자격 증명 스터핑, 서비스 거부 공격에 취약해집니다. 하지만 올바르게 구현하는 것은 생각보다 어렵습니다—특히 동시 요청이 관련될 때.

"작동하는" rate limiter가 부하 테스트에서 무제한 요청을 통과시키는 것을 발견하고 이를 뼈저리게 배웠습니다.

## 숨겨진 취약점

원래 구현은 합리적으로 보였습니다:

```typescript
async function checkRateLimit(userId: string): Promise<boolean> {
  const limits = loadRateLimits();           // 1. 현재 상태 읽기
  
  if (limits[userId] >= MAX_REQUESTS) {
    return false;                             // 2. 제한 확인
  }
  
  limits[userId] = (limits[userId] || 0) + 1; // 3. 증가
  saveRateLimits(limits);                     // 4. 다시 쓰기
  
  return true;
}
```

문제가 뭘까요? **TOCTOU** (Time-of-Check to Time-of-Use).

현재 카운트를 읽는 것(1단계)과 새 카운트를 쓰는 것(4단계) 사이에 다른 요청이 끼어들 수 있습니다:

```
시간 →
프로세스 A: Read(count=99) → Check(OK) → Increment → Write(100)
프로세스 B:      Read(count=99) → Check(OK) → Increment → Write(100)
프로세스 C:           Read(count=99) → Check(OK) → Increment → Write(100)
```

세 프로세스 모두 `count=99`를 읽고, 모두 검사를 통과하고, 모두 `count=100`을 씁니다. 하나만 통과해야 할 때 세 요청이 모두 통과합니다.

## 해결책: 원자적 연산

수정은 읽기-수정-쓰기 작업을 원자적으로 만들어야 합니다. 읽기와 쓰기 사이에 다른 프로세스가 상태에 접근할 수 없어야 합니다:

```typescript
import { Mutex } from 'async-mutex';

const rateLimitMutex = new Mutex();

async function checkRateLimit(userId: string): Promise<boolean> {
  return rateLimitMutex.runExclusive(async () => {
    const limits = loadRateLimits();
    
    if (limits[userId] >= MAX_REQUESTS) {
      return false;
    }
    
    limits[userId] = (limits[userId] || 0) + 1;
    saveRateLimits(limits);
    
    return true;
  });
}
```

이제 요청이 직렬화됩니다:

```
시간 →
프로세스 A: [LOCK] Read → Check → Increment → Write [UNLOCK]
프로세스 B:                                           [LOCK] Read → Check(FAIL!) [UNLOCK]
```

## 더 나은 대안들

프로덕션 시스템에서는 다음을 고려하세요:

1. **원자적 증가가 있는 Redis**: `INCR`은 본질적으로 원자적
   ```typescript
   const count = await redis.incr(`rate:${userId}`);
   await redis.expire(`rate:${userId}`, WINDOW_SECONDS);
   return count <= MAX_REQUESTS;
   ```

2. **데이터베이스 제약조건**: 데이터베이스가 원자성을 처리하도록
   ```sql
   UPDATE rate_limits 
   SET count = count + 1 
   WHERE user_id = ? AND count < 100;
   -- 영향받은 행 수를 확인하여 허용 여부 결정
   ```

3. **원자적 연산이 있는 토큰 버킷**: 버스트 허용이 있는 더 정교한 rate limiting

## 배운 점

1. **파일 기반 상태는 동시 접근에 위험** - 항상 여러 프로세스가 실행 중이라고 가정
2. **"읽기, 확인, 수정, 쓰기"는 위험 신호** - TOCTOU 취약점을 찾으세요
3. **동시성으로 테스트** - 단일 스레드 테스트는 레이스 컨디션을 잡지 못함
4. **보안 코드에는 보안 리뷰 필요** - 위협 모델링에 레이스 컨디션 분석 포함

이 취약점은 미묘했습니다—코드는 개발 환경에서 완벽하게 작동했지만 프로덕션 부하에서 실패했습니다. 레이스 컨디션은 당신이 보지 않을 때만 나타나는 타이밍 의존적 악마입니다.

## 예방 체크리스트

Rate limiting 배포 전:

- [ ] 증가 연산이 원자적인가?
- [ ] 동시 요청으로 테스트했는가 (`ab`, `wrk`, 또는 병렬 curl)?
- [ ] 적절한 데이터 저장소(Redis, 데이터베이스) 사용을 고려했는가?
- [ ] 코드베이스의 다른 TOCTOU 패턴을 검토했는가?

Rate limiter는 가장 약한 동시 접근 지점만큼만 강합니다.
