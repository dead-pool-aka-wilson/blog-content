---
title: "아무도 가리키지 않는 심링크"
date: 2026-02-14
author: Raoul
categories: [debugging]
tags: [symlink, path, migration, hugo]
draft: false
summary: "Hugo 블로그가 포스트 0개를 보여줬다. 콘텐츠는 있었다—이 머신에 존재하지 않는 사용자를 가리키고 있었을 뿐."
---

블로그가 설정됐다. Hugo가 에러 없이 실행됐다. 하지만 localhost:1313을 열었을 때, 포스트 섹션이 비어있었다.

콘텐츠가 있어야 한다는 걸 알았다. 포스트를 썼었다. 디렉토리 구조가 맞아 보였다. 하지만 Hugo는 아무것도 보지 못했다.

## 증상

```bash
hugo server
# 성공적으로 시작, 에러 없음
# 하지만: Posts: 0, Pages: 3

ls content/
# posts  skills  ...
# posts 디렉토리 존재!

ls content/posts/
# ... 아무것도 없음
```

`posts` 디렉토리가 존재하지만 비어있었다. 아니었나?

## 조사

```bash
ls -la content/posts
# lrwxr-xr-x  content/posts -> /Users/olduser/projects/blog/published
```

심링크. `/Users/olduser/...`를 가리키는.

하지만 나는 `olduser`가 아니다. `newuser`다. 이 머신에 `/Users/olduser` 디렉토리가 없다.

심링크는 다른 머신, 다른 시간, 다른 사용자 계정에서 온 유령 참조였다. Hugo가 링크를 따라가서 아무것도 찾지 못하고, 조용히 0개의 포스트를 보여줬다.

## 왜 이런 일이 일어나는가

절대 경로가 있는 심링크는 머신에 특정적이다. 인코딩하는 것:
- 전체 파일시스템 경로
- 사용자명 (홈 디렉토리 경로에서)
- 머신의 디렉토리 구조

레포를 클론하거나 닷파일을 새 머신에 복사할 때, 이 경로들은 업데이트되지 않는다. 심링크는 여전히 존재하지 않는 옛 위치를 가리킨다.

일반적인 시나리오:
- **닷파일 레포** - 설정이 종종 사용자 특정 경로에 심링크
- **모노레포 설정** - 패키지 간 심링크가 절대 경로 사용
- **백업 복원** - 옛 머신의 경로가 심링크에 내장
- **Docker 볼륨** - 호스트 경로가 컨테이너 설정에 굳어짐

## 해결

```bash
# 깨진 심링크 제거
rm content/posts

# 올바른 경로로 새 심링크 생성
ln -s "$HOME/Dev/blog-content/published" content/posts

# 확인
ls content/posts/
# 2026-02-02-ai-blog-series-1.md  ...
```

이 머신에 존재하지 않는 대상 디렉토리도 생성해야 했다:

```bash
mkdir -p "$HOME/Dev/blog-content/published"
```

### 깨진 심링크 진단

심링크 문제를 찾고 검증하는 빠른 명령:

```bash
# 현재 디렉토리의 모든 심링크 찾기
find . -type l -exec ls -la {} \;

# 구체적으로 깨진 심링크 찾기
find . -type l ! -exec test -e {} \; -print

# 특정 심링크가 무엇을 가리키는지 확인
readlink -f content/posts

# 심링크 대상이 존재하는지 테스트
test -e "$(readlink content/posts)" && echo "OK" || echo "BROKEN"
```

## 예방 전략

### 상대 심링크 사용
가능하면 링크 위치에 상대적인 심링크 생성:

```bash
# 이것 대신:
ln -s /Users/koed/Dev/BrainFucked/10-Blog/published content/posts

# 이걸 사용:
cd content
ln -s ../../BrainFucked/10-Blog/published posts
```

상대 심링크는 디렉토리 이동에서 살아남는다 (상대 위치가 같은 한).

### 클론 후 심링크 확인
"새 머신 설정" 체크리스트에 추가:

```bash
# 레포의 모든 심링크 찾기
find . -type l -exec ls -la {} \;

# 깨진 심링크 확인
find . -type l ! -exec test -e {} \; -print
```

### 스크립트에서 환경 변수 사용
심링크가 반드시 절대여야 한다면, 환경에서 생성:

```bash
ln -s "${HOME}/Dev/BrainFucked/10-Blog/published" content/posts
```

이건 최소한 현재 사용자에 맞춰진다.

### 경로 의존성 문서화
README에 클론 후 업데이트 필요한 경로 나열:

```markdown
## 클론 후 설정
당신의 머신에 맞게 심링크 업데이트:
- `content/posts` → 당신의 `published/` 디렉토리
```

## 더 넓은 교훈

버그는 조용했다. 에러 없음, 경고 없음. Hugo가 빈 디렉토리를 보고 계속했다. 있어야 할 콘텐츠가 없었기에만 알아챘다.

조용한 실패가 최악의 종류다. 깨진 심링크는 예외를 던지지 않는다—그냥 아무것도 아닌 것으로 해결된다.

머신 마이그레이션 후 뭔가 "사라지면":
1. 심링크 먼저 확인: `ls -la`가 대상을 보여줌
2. 대상 존재 확인: `test -e $(readlink file)`
3. 절대 경로 찾기: `/Users/`나 `/home/`로 시작하는 것

배관이 코드가 실행되기 전에 실패했다. 때로는 디버깅이 코드가 아니라—그 아래의 파일시스템에 관한 것이다.
