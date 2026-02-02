---
date: 2026-02-02
project: opencode-git-workflow
session: ses_3e3ced5d5ffe5qCJE4byJIGynF
tags: [ultrawork, multi-feature, git-workflow, plugin-development]
content_potential: high
---

## The Prompt

```
add checking pr review and commit limit per pr feature. add prompt information in issue and add prompt and response information in pr. ultrawork
```

## Context

User wanted to enhance an existing OpenCode Git Workflow Enforcer plugin with 4 distinct features:
1. PR review checking (block merge if not approved)
2. Commit limit per PR enforcement (block PR creation if too many commits)
3. Enhanced prompt info in GitHub issues (full history)
4. Prompt + response info in PRs (conversation context)

The `ultrawork` suffix triggered a specific mode requiring:
- Plan agent invocation before implementation
- Parallel task execution in waves
- Full verification before completion

## Outcome

**Worked** - All 4 features implemented in ~35 minutes:
- 4 commits, 1 PR created
- ~255 lines added to plugin
- Documentation updated across 3 files

## Notable Because

1. **Concise multi-feature request**: 4 distinct features in one sentence, no ambiguity
2. **Implicit scope clarity**: Each feature had clear boundaries
3. **Mode trigger**: `ultrawork` suffix activated specific behavior (plan-first, parallel execution)
4. **Zero clarification needed**: Request was specific enough to proceed directly

## Pattern

```
{feature1} + {feature2} + {feature3} + {feature4}. {mode-trigger}
```

When features are:
- Independent (can be implemented separately)
- Clear in scope (no ambiguity)
- Related to same codebase

This pattern allows efficient batch implementation vs 4 separate requests.
