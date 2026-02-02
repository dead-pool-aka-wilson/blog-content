---
name: project-journal
description: Generate project status update blog posts. Activate when user wants to document project progress, milestones, or development logs. 프로젝트 상태 업데이트 블로그 포스트 생성. 프로젝트 진행 상황, 마일스톤, 개발 로그 문서화 시 활성화.
---

# Project Journal Generator

프로젝트 진행 상황을 문서화하고 블로그 포스트로 변환합니다.

## When to Activate

- User says "프로젝트 업데이트" / "project update"
- User wants to document milestone completion
- User requests development log post / 개발 로그 포스트 요청
- Weekly/monthly project summary needed
- User says "이번 주 작업 정리해줘"

## Update Types

### 1. Progress Update (진행 상황)
**Use when:** Regular status check (weekly/bi-weekly)
- What was accomplished this period
- Metrics/KPIs changes
- Visual evidence of progress

### 2. Milestone Post (마일스톤)
**Use when:** Significant achievement reached
- What the milestone represents
- Journey to get there
- Lessons learned along the way

### 3. Development Log (개발 로그)
**Use when:** Technical deep-dive needed
- Technical decisions made and why
- Problems encountered
- Solutions implemented
- Code examples

### 4. Retrospective (회고)
**Use when:** Period review (monthly/quarterly/project end)
- What went well
- What could improve
- Action items for next period

## Hugo Output Template

### Frontmatter

```yaml
---
title: "[Project Name]: [Update Type] - [Period/Milestone]"
date: YYYY-MM-DD
categories: [project-journal, development-log, milestone]
tags: [project-name, update-type]
draft: false
lang: ko  # or en
project:
  name: "[Project Name]"
  status: active/paused/completed
  period: "YYYY-MM-DD to YYYY-MM-DD"
  github: "[repo URL if applicable]"
---
```

### Korean Post Structure

```markdown
## TL;DR
- [핵심 요약 1]
- [핵심 요약 2]
- [핵심 요약 3]

## 프로젝트 상태

| 항목 | 내용 |
|------|------|
| **프로젝트** | [이름] |
| **기간** | [시작일 - 종료일] |
| **상태** | 🟢 정상 진행 / 🟡 지연 / 🔴 중단 |

## 이번 기간 완료 항목

### ✅ 완료
- [x] [완료 항목 1]
- [x] [완료 항목 2]
- [x] [완료 항목 3]

### 📊 수치로 보기

| 지표 | 이전 | 현재 | 변화 |
|------|------|------|------|
| [지표1] | X | Y | +Z% |
| [지표2] | X | Y | +Z% |
| [지표3] | X | Y | +Z% |

## 스크린샷 / 시각 자료

![설명](../images/posts/YYYY-MM-DD-project-update/screenshot-1.png)

## 기술적 결정사항

### [결정 1: 제목]
- **맥락:** [왜 이 결정이 필요했는지]
- **선택지:** [고려한 옵션들]
- **결정:** [선택한 것과 이유]
- **결과:** [결정의 영향]

## 겪은 문제와 해결

### 문제 1: [문제 설명]
- **증상:** [어떤 일이 발생했는지]
- **원인:** [근본 원인]
- **해결:** [어떻게 해결했는지]
- **교훈:** [배운 점]

## 다음 단계

### 📋 다음 기간 목표
- [ ] [할 일 1]
- [ ] [할 일 2]
- [ ] [할 일 3]

### ⏰ 예상 일정
[다음 마일스톤까지 예상 시간]

## 도움 필요 / 블로커
[현재 막힌 부분이나 도움이 필요한 사항]

---

*다음 업데이트: [예정일]*
```

### English Post Structure

```markdown
## TL;DR
- [Key summary 1]
- [Key summary 2]
- [Key summary 3]

## Project Status

| Item | Details |
|------|---------|
| **Project** | [Name] |
| **Period** | [Start - End] |
| **Status** | 🟢 On Track / 🟡 Delayed / 🔴 Blocked |

## Completed This Period

### ✅ Done
- [x] [Completed item 1]
- [x] [Completed item 2]
- [x] [Completed item 3]

### 📊 By the Numbers

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| [Metric1] | X | Y | +Z% |
| [Metric2] | X | Y | +Z% |
| [Metric3] | X | Y | +Z% |

## Screenshots / Visuals

![description](../images/posts/YYYY-MM-DD-project-update/screenshot-1.png)

## Technical Decisions

### [Decision 1: Title]
- **Context:** [Why this decision was needed]
- **Options:** [What was considered]
- **Choice:** [What was chosen and why]
- **Impact:** [Results of the decision]

## Problems & Solutions

### Problem 1: [Description]
- **Symptom:** [What happened]
- **Root Cause:** [Why it happened]
- **Solution:** [How it was fixed]
- **Lesson:** [What was learned]

## Next Steps

### 📋 Goals for Next Period
- [ ] [Todo 1]
- [ ] [Todo 2]
- [ ] [Todo 3]

### ⏰ Timeline
[Expected time to next milestone]

## Help Needed / Blockers
[Current blockers or areas needing assistance]

---

*Next update: [scheduled date]*
```

## Screenshot Integration

**Use playwright skill to capture:**
- UI before/after comparisons
- Terminal outputs showing progress
- Architecture diagrams
- Metric dashboards
- Error messages (for problem documentation)

```
/playwright - capture project visuals:
- Current UI state
- Performance dashboards
- Test results
- Deployment status
```

**Save to:** `static/images/posts/YYYY-MM-DD-project-update/`

## Status Indicators

| Indicator | Korean | English | Use When |
|-----------|--------|---------|----------|
| 🟢 | 정상 진행 | On Track | Everything proceeding as planned |
| 🟡 | 지연 | Delayed | Behind schedule but recoverable |
| 🔴 | 중단/블록 | Blocked | Cannot proceed without help |
| ⏸️ | 일시 중지 | Paused | Intentionally stopped |
| ✅ | 완료 | Completed | Project finished |

## Checklist

- [ ] Project context clear / 프로젝트 맥락 명확
- [ ] Status indicator set / 상태 표시 설정
- [ ] Progress documented with evidence / 진행 상황 증거와 함께 기록
- [ ] Metrics included where applicable / 해당되는 경우 지표 포함
- [ ] Technical decisions explained / 기술적 결정 설명
- [ ] Problems and solutions documented / 문제와 해결책 문서화
- [ ] Screenshots captured via playwright / Playwright로 스크린샷 캡처
- [ ] Next steps defined / 다음 단계 정의
- [ ] Blockers identified if any / 블로커 식별 (있는 경우)
- [ ] Next update date set / 다음 업데이트 날짜 설정

---

**Remember**: Project journals are not just for others—they're a gift to your future self. Document decisions and context that you'll forget in 6 months.
