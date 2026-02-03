---
title: "AI 봇 샌드박스를 위한 안전한 테스트: 단계별 접근"
date: 2026-02-10
author: Raoul
categories: [technical]
tags: [docker, sandbox, security, testing, automation]
draft: false
summary: "AI 봇이 문자를 보내고 실제 파일에 접근할 수 있을 때, 실제 데이터를 날리지 않고 샌드박스를 어떻게 테스트하나? 4단계 접근법."
cover:
  image: "/covers/cover-2026-02-10-phased-docker-sandbox-testing.png"
---

봇이 내 폰의 Signal 신원과 Obsidian 볼트에 접근할 수 있었다. 샌드박스가 실제로 그것을 담고 있는지 어떻게 테스트하나?

Mac Mini의 Docker 컨테이너 안에 moltbot—개인 자동화 봇—을 설정하고 있었다. 봇은 Signal로 알림을 보내고, 노트 볼트를 읽고 쓰고, 이메일에 접근해야 한다. 각 기능은 리스크이기도 하다: 잘못된 설정이 Signal 신원을 망가뜨리거나, 노트를 손상시키거나, 의도하지 않은 이메일을 보낼 수 있다.

Docker 격리가 이를 방지해야 한다. 하지만 실제 데이터가 걸려 있을 때 "해야 한다"는 충분하지 않다.

## WHY: 신뢰하되 검증하라

Docker 리소스 제한과 볼륨 마운트가 설정 파일에서는 맞아 보인다. 하지만 테스트하기 전까지 모른다:

- CPU 제한이 실제로 컨테이너를 제약하나?
- 컨테이너가 마운트된 볼륨 밖에 쓸 수 있나?
- 읽기 전용 플래그가 실제로 쓰기를 방지하나?

설정은 무엇이 일어나야 하는지 말한다. 테스트는 무엇이 일어나는지 확인한다.

## HOW: 샌드박스 테스트의 4단계

### 1단계: 더미 데이터로 격리된 컨테이너

절대 실제 데이터로 먼저 테스트하지 마라. 가짜 마운트 포인트 생성:

```bash
# 더미 콘텐츠로 테스트 디렉토리 생성
mkdir -p /tmp/test-sandbox/{downloads,obsidian}
echo "test note" > /tmp/test-sandbox/obsidian/test.md

# 실제 경로가 아닌 더미 마운트로 컨테이너 실행
docker run --rm -it \
  --read-only \
  --cpus="2.0" \
  --memory="2g" \
  -v /tmp/test-sandbox/downloads:/mnt/downloads:rw \
  -v /tmp/test-sandbox/obsidian:/mnt/obsidian:ro \
  moltbot-sandbox:latest /bin/sh
```

컨테이너 안에서 시도:
- 읽기 전용 마운트에 쓰기 (실패해야 함)
- 마운트된 볼륨 밖에 쓰기 (실패해야 함)
- 마운트된 경로에서 읽기 확인

이 중 예상대로 작동하지 않는 게 있으면, 실제 데이터 건드리기 전에 설정 수정.

### 2단계: 리소스 제한 검증

설정에 `--cpus="2.0"`과 `--memory="2g"`가 있다. 실제로 제한하나?

```bash
docker run --rm -it \
  --cpus="2" \
  --memory="2g" \
  moltbot-sandbox:latest sh -c "
    # 8 CPU 사용 시도 - 2로 쓰로틀되어야 함
    stress-ng --cpu 8 --timeout 10s 2>/dev/null && echo 'Completed' || echo 'Expected limitation'
    
    # 보이는 CPU 수 확인
    nproc
    
    # 메모리 제한 확인
    cat /sys/fs/cgroup/memory/memory.limit_in_bytes 2>/dev/null || \
    cat /sys/fs/cgroup/memory.max 2>/dev/null
  "
```

스트레스 테스트 실행 중 다른 터미널에서 `docker stats` 관찰. CPU 사용량이 제한에서 막혀야 한다.

### 3단계: 자격 증명 테스트 (읽기 전용 작업)

전송 작업 테스트 전에, 실제 자격 증명으로 읽기 작업이 동작하는지 확인:

```bash
# 이메일: 읽기 전용 작업만
himalaya list --folder INBOX --page-size 3  # 안전: 목록만
himalaya read 1 --headers  # 안전: 하나만 읽기

# 테스트 중 절대 안 됨:
# himalaya send ...
# himalaya forward ...
```

읽기 작업은 의도하지 않은 액션의 위험 없이 자격 증명이 작동함을 증명한다. 샌드박스가 제대로 담고 있는지 확인한 후에만 쓰기 작업 진행.

### 4단계: 고위험 컴포넌트 격리

어떤 컴포넌트는 가볍게 테스트하기엔 너무 위험하다. 예를 들어 Signal-cli는 폰의 신원을 관리한다. 잘못된 설정이 망가뜨릴 수 있다.

**Signal 테스트 전:**
```bash
# 신원 완전 백업
cp -r ~/.signal-cli ~/.signal-cli.backup.$(date +%Y%m%d)
```

**테스트 중:**
- 가능하면 버너 번호 사용
- 수신 전용 작업으로 시작
- 자신에게 먼저 전송 테스트
- 다른 모든 것이 작동할 때까지 그룹 작업 테스트 안 함

**뭔가 망가지면:**
```bash
# 백업에서 복원
rm -rf ~/.signal-cli
cp -r ~/.signal-cli.backup.$(date +%Y%m%d) ~/.signal-cli
```

## WHAT: 리스크 매트릭스

테스트 전에 각 연동을 리스크로 분류:

| 연동 | 읽기 리스크 | 쓰기 리스크 | 복구 |
|------|-------------|-------------|------|
| 파일 마운트 | 낮음 | 중간 | 백업에서 복원 |
| 이메일 | 낮음 | 높음 | 보이지만 어색함 |
| Signal | 중간 | 치명적 | 백업 필수 |
| 캘린더 | 낮음 | 중간 | 이벤트 삭제 가능 |
| 데이터베이스 | 중간 | 치명적 | 백업 필수 |

**높은 쓰기 리스크** 연동은 추가 단계가 있다:
1. 목/스텁으로 테스트
2. 읽기 작업만 테스트
3. 자신/테스트 대상에게 쓰기 테스트
4. 백업 준비 후 실제 작업 테스트

### 자동화 테스트 체크리스트

어떤 샌드박스 설정도 "테스트됨"으로 간주하기 전에:

- [ ] 컨테이너가 정의된 마운트 밖에 쓸 수 없음
- [ ] 읽기 전용 마운트가 실제로 쓰기 방지
- [ ] 리소스 제한이 강제됨 (설정만이 아닌)
- [ ] 설정된 경우 네트워크 격리 작동
- [ ] 읽기 작업에 자격 증명 작동
- [ ] 고위험 연동에 백업 존재
- [ ] 전송/쓰기 작업을 안전한 대상으로 먼저 테스트

## 요약

질문은 "Docker 샌드박스가 작동할까?"가 아니었다. "실제 데이터를 파괴하지 않으면서 작동함을 어떻게 증명하나?"였다.

단계별 테스트가 답이다. 가짜 데이터로 시작. 제약이 실제로 제약하는지 확인. 쓰기 전에 읽기 테스트. 고위험 작업 전에 모든 중요한 것 백업.

내 moltbot은 이제 올바르게 설정되었길 바라는 게 아니라 실제로 검증한 샌드박스에서 실행된다. 단계별 테스트에 쓴 한 시간이 "괜찮을 줄 알았는데" 순간에서 복구하느라 쓸 하루를 이긴다.

봇이 실제 데이터에 접근할 때, 희망은 보안 전략이 아니다. 신뢰하기 전에 우리를 테스트하라.
