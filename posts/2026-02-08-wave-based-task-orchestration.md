---
title: "Wave-Based Task Orchestration: Parallelizing AI Agent Work"
date: 2026-02-08
author: Raoul
categories: [technical]
tags: [ai-orchestration, parallel-execution, task-planning, dependency-management]
draft: false
summary: "Breaking complex tasks into dependency-aware waves lets multiple AI agents work simultaneously—but theoretical parallelism doesn't always match practical execution."
lang: en
cover:
  image: "/covers/cover-2026-02-08-wave-based-task-orchestration.png"
---

How do you coordinate multiple AI agents working on the same codebase without them stepping on each other?

I've been experimenting with a plan-then-execute pattern for complex implementations. Instead of having one agent work through tasks sequentially, I break the work into "waves"—groups of tasks that can theoretically run in parallel because they don't depend on each other.

The results are interesting. The speedup is real, but the gap between theoretical and practical parallelism taught me something about how to think about AI coordination.

## WHY: Sequential Is Slow

Consider a feature that requires seven related changes:
1. Extend config schema (foundation)
2. Add event hook system
3. Add review checking logic
4. Add commit enforcement
5. Enhance issue creation tool
6. Add PR creation tool
7. Update documentation

Done sequentially, you wait for each task to complete before starting the next. If each takes 10 minutes, that's 70 minutes of wall-clock time—even if tasks 2, 3, and 4 could have run simultaneously.

The insight: most multi-task implementations have a dependency structure that allows parallelism. Finding that structure is the key to faster execution.

## HOW: Mapping Dependencies to Waves

A "wave" is a set of tasks that can run concurrently. Tasks in the same wave have no dependencies on each other—they can all start as soon as the previous wave completes.

Here's how a plan agent structured the seven-task example:

```
Wave 1 (Start Immediately):
└── Task 1: Extend config schema [no dependencies]

Wave 2 (After Wave 1):
├── Task 2: Add event hook [depends: 1]
├── Task 3: Add review checking [depends: 1]
└── Task 4: Add commit enforcement [depends: 1]

Wave 3 (After Wave 2):
├── Task 5: Enhance issue tool [depends: 2]
└── Task 6: Add PR tool [depends: 2, 3, 4]

Wave 4 (After Wave 3):
└── Task 7: Update documentation [depends: 5, 6]

Critical Path: Task 1 → Task 2 → Task 6 → Task 7
Estimated Speedup: ~35% faster than sequential
```

The critical path is the longest chain of dependent tasks. No matter how much you parallelize, you can't go faster than the critical path. But everything not on the critical path can potentially run simultaneously with it.

### Building the Dependency Graph

For each task, ask: "What must be complete before this can start?"

Common dependency types:
- **Schema/type dependencies**: Can't use a new config field before it's defined
- **API dependencies**: Can't call a function before it exists
- **Data dependencies**: Can't process output before input is generated
- **Test dependencies**: Can't verify integration before components exist

Tasks with no dependencies go in Wave 1. Tasks that only depend on Wave 1 tasks go in Wave 2. And so on.

## WHAT: Theory vs. Practice

The plan looked perfect. Wave 2 had three independent tasks—event hooks, review checking, and commit enforcement. In theory, three agents could work simultaneously.

In practice, all three tasks modified the same file: `git-workflow-enforcer.ts`.

**Theoretical parallelism**: Tasks are logically independent (different features)
**Practical parallelism**: Tasks have file-level conflicts

When multiple agents edit the same file simultaneously, you get merge conflicts, overwritten changes, or incoherent code. The execution layer correctly serialized Wave 2 tasks even though the plan marked them as parallel.

### The File Conflict Problem

This is the gap between dependency planning and execution reality:

| Dependency Level | What It Means | Example |
|------------------|---------------|---------|
| Feature dependencies | Task B uses output of Task A | PR tool needs issue numbers |
| API dependencies | Task B calls function from Task A | Hook system must exist first |
| File dependencies | Task A and B edit same file | Both add methods to same class |

Plan agents typically model feature and API dependencies. File dependencies require deeper analysis—knowing which files each task will touch before execution.

### Mitigations

**Conservative parallelism**: When unsure, serialize. The overhead of occasional unnecessary serialization is lower than the cost of merge conflict resolution.

**File-aware planning**: Include "touches files" in task specifications. Group tasks that modify the same files into the same wave, executed sequentially.

**Conflict detection**: Before starting a parallel wave, check if tasks target overlapping files. If yes, demote to sequential within that wave.

**Smaller files**: If `git-workflow-enforcer.ts` were split into `event-hooks.ts`, `review-checking.ts`, and `commit-enforcement.ts`, Wave 2 would have been genuinely parallel.

### Actual Speedup

Despite the serialization of Wave 2, the overall execution was still faster than pure sequential:

- Waves 3 and 4 ran as planned
- Parallel exploration (gathering context) happened before task execution
- Agent startup overhead was amortized across fewer wait cycles

The 35% theoretical speedup became roughly 20% actual speedup. Still worth doing.

## The Pattern

Wave-based orchestration works, but requires honesty about what's actually parallel:

1. **Map dependencies** - Build a graph of what depends on what
2. **Group into waves** - Tasks with satisfied dependencies run together
3. **Check for conflicts** - File-level, state-level, test-level
4. **Execute with fallbacks** - Serialize when conflicts detected
5. **Measure actual speedup** - Theory rarely matches practice

The biggest lesson: dependency analysis is cheap; conflict resolution is expensive. Over-parallelize in the plan, but have the execution layer ready to serialize when reality disagrees.

For complex multi-agent work, this wave pattern has become my default. It forces explicit dependency thinking, provides natural checkpoints between waves, and surfaces parallelism opportunities that sequential planning misses.

Just don't trust the theoretical speedup numbers until you've seen the file diff.
