---
title: "TOCTOU Race Conditions in Rate Limiting: A Security Deep Dive"
date: 2026-02-04
categories: [security, debugging]
tags: [toctou, race-condition, rate-limiting, concurrency, security]
draft: false
summary: "How a subtle race condition in file-based rate limiting allowed unlimited API calls, and the atomic operations that fixed it."
lang: en
cover:
  image: "/covers/2026-02-04-toctou-rate-limiting.png"
---

## Why This Matters

Rate limiting is a critical security control. When it fails, your API becomes vulnerable to abuse, credential stuffing, and denial of service attacks. But implementing it correctly is harder than it looks—especially when concurrent requests are involved.

I discovered this the hard way when my "working" rate limiter let through unlimited requests under load.

## The Hidden Vulnerability

The original implementation looked reasonable:

```typescript
async function checkRateLimit(userId: string): Promise<boolean> {
  const limits = loadRateLimits();           // 1. Read current state
  
  if (limits[userId] >= MAX_REQUESTS) {
    return false;                             // 2. Check limit
  }
  
  limits[userId] = (limits[userId] || 0) + 1; // 3. Increment
  saveRateLimits(limits);                     // 4. Write back
  
  return true;
}
```

The problem? **TOCTOU** (Time-of-Check to Time-of-Use). 

Between reading the current count (step 1) and writing the new count (step 4), another request can slip through:

```
Time →
Process A: Read(count=99) → Check(OK) → Increment → Write(100)
Process B:      Read(count=99) → Check(OK) → Increment → Write(100)
Process C:           Read(count=99) → Check(OK) → Increment → Write(100)
```

All three processes read `count=99`, all pass the check, and all write `count=100`. Three requests go through when only one should.

## The Solution: Atomic Operations

The fix requires making the read-modify-write operation atomic. No other process can access the state between our read and write:

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

Now requests are serialized:

```
Time →
Process A: [LOCK] Read → Check → Increment → Write [UNLOCK]
Process B:                                           [LOCK] Read → Check(FAIL!) [UNLOCK]
```

## Better Alternatives

For production systems, consider:

1. **Redis with atomic increment**: `INCR` is inherently atomic
   ```typescript
   const count = await redis.incr(`rate:${userId}`);
   await redis.expire(`rate:${userId}`, WINDOW_SECONDS);
   return count <= MAX_REQUESTS;
   ```

2. **Database constraints**: Let the database handle atomicity
   ```sql
   UPDATE rate_limits 
   SET count = count + 1 
   WHERE user_id = ? AND count < 100;
   -- Check affected rows to determine if allowed
   ```

3. **Token bucket with atomic operations**: More sophisticated rate limiting with burst allowance

## Lessons Learned

1. **File-based state is dangerous for concurrent access** - Always assume multiple processes are running
2. **"Read, check, modify, write" is a red flag** - Look for TOCTOU vulnerabilities
3. **Test with concurrency** - A single-threaded test won't catch race conditions
4. **Security code needs security review** - Include race condition analysis in threat modeling

The vulnerability was subtle—the code worked perfectly in development but failed under production load. Race conditions are timing-dependent demons that only appear when you're not looking.

## Prevention Checklist

Before deploying rate limiting:

- [ ] Is the increment operation atomic?
- [ ] Tested with concurrent requests (`ab`, `wrk`, or parallel curl)?
- [ ] Considered using a proper data store (Redis, database)?
- [ ] Reviewed for other TOCTOU patterns in the codebase?

Your rate limiter is only as strong as its weakest concurrent access point.
