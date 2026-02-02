---
date: 2026-02-02
session: ses_3e3ced5d5ffe5qCJE4byJIGynF
tags: [ai-planning, task-structure, delegation, prompt-engineering]
stage: seed
content_potential: medium
---

## The Idea

A structured task definition template that ensures AI agents have everything needed for autonomous execution.

## Context

The plan agent produced tasks with consistent structure that enabled clean delegation:

```markdown
### Task N: [Name]

**What to do**:
- Specific actions with code examples

**Must NOT do**:
- Guardrails to prevent scope creep

**Recommended Agent Profile**:
- **Category**: `quick|visual-engineering|ultrabrain|...`
- **Skills**: [`skill-1`, `skill-2`]
- **Skills Evaluated but Omitted**: [with reasoning]

**Parallelization**:
- **Can Run In Parallel**: YES/NO
- **Parallel Group**: Wave N
- **Blocks**: Tasks X, Y
- **Blocked By**: Tasks A, B

**References**:
- File paths with line numbers
- External docs
- WHY each reference matters

**Acceptance Criteria**:
- Bash commands agent can run to verify
- Expected outputs

**Commit**: YES/NO
- Message format
- Files to include
- Pre-commit verification
```

## Raw Exchange

> Plan output included 7 tasks with this structure
> Each task was delegatable with zero ambiguity
> Verification commands were agent-executable (grep patterns, exit codes)

## Key Components

| Section | Purpose |
|---------|---------|
| What to do | Clear scope |
| Must NOT do | Prevent rogue behavior |
| Category + Skills | Right tool for job |
| References | Context without re-exploration |
| Acceptance Criteria | Self-verifiable completion |

## Expansion Potential

1. **Reusable template** for other planning scenarios
2. **Checklist** for evaluating AI-generated plans
3. **Comparison** with human task specifications
