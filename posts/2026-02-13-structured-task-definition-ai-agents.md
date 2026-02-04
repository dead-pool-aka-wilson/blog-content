---
title: "The Anatomy of a Delegatable Task: Structured Definitions for AI Agents"
date: 2026-02-13
author: Raoul
categories: [technical]
tags: [ai-planning, task-structure, delegation, prompt-engineering]
draft: false
summary: "Vague task descriptions produce vague results. A structured template ensures AI agents have everything needed for autonomous execution."
lang: en
cover:
  image: "/covers/cover-2026-02-13-structured-task-definition-ai-agents.png"
---

Why do some delegated tasks come back perfect while others need three rounds of clarification?

The difference isn't the AI's capability—it's the task definition. Vague instructions produce vague results. Structured definitions produce autonomous execution.

After watching a plan agent produce consistently delegatable tasks, I extracted the template. Every section exists because its absence causes problems.

## WHY: Ambiguity Is Expensive

When you delegate a task to an AI agent with "implement the login feature," you get:
- Questions about scope ("Does this include password reset?")
- Wrong assumptions about patterns ("I used JWT because...")
- Missing edge cases ("I didn't add rate limiting because...")
- Unverifiable completion ("It should work now")

Each clarification round costs time. Each wrong assumption costs rework. The upfront investment in task structure pays for itself in reduced iteration.

## HOW: The Template

A complete task definition has seven sections:

```markdown
### Task N: [Descriptive Name]

**What to do**:
- Specific actions with code examples
- Expected changes with file paths
- Technical approach to use

**Must NOT do**:
- Explicit guardrails
- Forbidden patterns
- Scope boundaries

**Recommended Agent Profile**:
- **Category**: `quick|visual-engineering|ultrabrain|deep|writing`
- **Skills**: [`skill-1`, `skill-2`]
- **Skills Evaluated but Omitted**: [with reasoning]

**Parallelization**:
- **Can Run In Parallel**: YES/NO
- **Parallel Group**: Wave N
- **Blocks**: Tasks X, Y
- **Blocked By**: Tasks A, B

**References**:
- `path/to/file.ts:42-60` - specific lines
- External documentation links
- WHY each reference matters

**Acceptance Criteria**:
```bash
# Commands agent can run to verify
grep -q "expected_pattern" file.ts && echo "PASS"
npm test -- --grep "feature"
```

**Commit**: YES/NO
- Message format
- Files to include
```

### Why Each Section Matters

**What to do**: Removes ambiguity about scope. "Implement login" vs "Add POST /api/login endpoint that validates email/password against users table and returns JWT with 24h expiry" are different levels of specification.

**Must NOT do**: Prevents scope creep and rogue behavior. Agents optimize for completion—they'll add features you didn't ask for if not constrained. "Do not add password reset flow" is explicit.

**Category + Skills**: Routes to the right capability. A UI task shouldn't go to a backend-specialized agent. Skills inject domain knowledge.

**Parallelization**: Enables concurrent execution. Without dependency information, tasks must run sequentially. With it, independent tasks can parallelize.

**References**: Provides context without re-exploration. Instead of "look at how auth is done elsewhere," provide exact files and lines. The agent doesn't waste time searching.

**Acceptance Criteria**: Makes completion verifiable. "It should work" vs "running `npm test auth` passes" is the difference between "trust me" and "prove it."

**Commit**: Specifies output expectations. Should the agent commit? With what message? What files should be included?

## WHAT: Examples

### Bad Task Definition
```
Task: Add user authentication

Do: Implement login and signup
References: Look at existing auth code
```

Problems:
- No technical approach specified
- No scope boundaries
- "Look at" requires exploration
- No verification criteria

### Good Task Definition
```markdown
### Task 3: Add Login Endpoint

**What to do**:
- Create `POST /api/auth/login` in `src/routes/auth.ts`
- Validate request body: `{ email: string, password: string }`
- Check credentials against `users` table via Prisma
- Return JWT with 24h expiry using existing `signToken()` helper
- Return 401 for invalid credentials with message "Invalid email or password"

**Must NOT do**:
- Do not add signup endpoint (Task 4)
- Do not add password reset
- Do not modify existing auth middleware
- Do not add rate limiting (separate task)

**Recommended Agent Profile**:
- **Category**: `quick`
- **Skills**: [`typescript-programmer`]

**Parallelization**:
- **Can Run In Parallel**: NO
- **Blocked By**: Task 2 (database schema)

**References**:
- `src/routes/auth.ts:1-30` - existing auth structure
- `src/utils/jwt.ts:15-25` - signToken helper
- `prisma/schema.prisma:42-50` - User model
- WHY: Shows existing patterns to match

**Acceptance Criteria**:
```bash
# Endpoint exists
grep -q "router.post('/login'" src/routes/auth.ts

# Returns JWT on valid creds
curl -X POST localhost:3000/api/auth/login \
  -d '{"email":"test@test.com","password":"test"}' \
  | jq -e '.token'

# Returns 401 on invalid creds  
curl -s -o /dev/null -w "%{http_code}" \
  -X POST localhost:3000/api/auth/login \
  -d '{"email":"wrong","password":"wrong"}' | grep -q "401"
```

**Commit**: YES
- Message: `feat(auth): add login endpoint`
- Files: `src/routes/auth.ts`
```

This task can be delegated and executed autonomously. The agent knows exactly what to do, what not to do, where to look, and how to verify completion.

## The Takeaway

Every section in the template exists because I've seen tasks fail without it:
- Missing "what to do" → wrong scope
- Missing "must not do" → scope creep
- Missing references → wasted exploration
- Missing acceptance criteria → unverifiable completion

Investing time in task structure front-loads the thinking. The agent executes; you don't babysit.

A task definition that takes 5 minutes to write saves 30 minutes of clarification and rework. Five minutes of structure beats thirty minutes of back-and-forth.
