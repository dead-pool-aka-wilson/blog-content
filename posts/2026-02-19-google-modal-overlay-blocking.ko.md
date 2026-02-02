---
title: "보이지 않는 모달: Google UI에서 Playwright 클릭 디버깅"
date: 2026-02-19
author: Raoul
categories: [debugging]
tags: [playwright, google-ui, automation, modal]
draft: false
summary: "버튼이 보였다. 셀렉터가 찾았다. 클릭은 아무것도 안 했다. 보이지 않는 모달 오버레이가 모든 상호작용을 훔치고 있었다."
---

"Add source 버튼 클릭 불가."

스크린샷에서 버튼을 볼 수 있었다. Playwright가 엘리먼트를 찾았다—셀렉터가 작동했다. 하지만 `click()`은 조용히 실패하거나 타임아웃됐다. 버튼은 존재했다. 그냥 반응하지 않았다.

## 증상

Playwright로 NotebookLM 업로드 자동화:

```typescript
// 버튼 찾기 - 성공
const addButton = await page.locator('button[aria-label="Add source"]');
await expect(addButton).toBeVisible();  // 통과!

// 버튼 클릭 - 실패
await addButton.click();  // 타임아웃 또는 조용한 실패
```

엘리먼트를 찾았다. 가시성 체크 통과. 클릭이 작동하지 않았다.

## 조사

Playwright의 `click()`은 클릭 이벤트 트리거 이상을 한다:
1. 엘리먼트를 뷰로 스크롤
2. 엘리먼트가 안정될 때까지 대기
3. 엘리먼트가 액션 가능한지 확인 (보이고, 활성화되고, 가려지지 않음)
4. 엘리먼트 중앙 클릭

4단계는 뭔가가 엘리먼트를 덮고 있으면 실패할 수 있다—그 뭔가가 보이지 않더라도.

클릭 전에 디버그 스크린샷 추가가 문제를 드러냈다:

```typescript
await page.screenshot({ path: 'debug-before-click.png' });
```

스크린샷이 전체 페이지를 덮는 반투명 오버레이를 보여줬다. Google이 검색 다이얼로그, 동의 프롬프트, 또는 프로모 모달을 열었다. 오버레이는 시각적으로 미묘했지만 모든 클릭 이벤트를 캡처했다.

## 해결

상호작용 전에 모달을 클리어하는 `dismissOverlays()` 함수 생성:

```typescript
async function dismissOverlays(page: Page): Promise<void> {
  // 열린 다이얼로그 닫기 위해 Escape 여러 번 누르기
  for (let i = 0; i < 3; i++) {
    await page.keyboard.press("Escape");
    await page.waitForTimeout(200);
  }
  
  // 모달 영역 밖 클릭 (왼쪽 상단 코너)
  try {
    await page.click("body", { 
      position: { x: 0, y: 0 }, 
      force: true 
    });
  } catch {
    // 클릭 실패 무시 - body가 클릭 불가능할 수 있음
  }
  
  // 애니메이션 정착 대기
  await page.waitForTimeout(300);
}
```

중요한 상호작용 전 사용:

```typescript
await dismissOverlays(page);
await page.locator('button[aria-label="Add source"]').click();
```

### 왜 여러 번 Escape?

Google UI는 중첩 모달이 있을 수 있다:
- 검색 오버레이
- 그 아래 동의 다이얼로그
- 그 아래 온보딩 툴팁

한 번 Escape가 최상위 레이어를 닫고 다른 것을 드러낼 수 있다. 세 번 Escape가 합리적인 중첩을 처리한다.

### 왜 `force: true`?

`force: true` 옵션은 Playwright의 액션 가능성 체크를 우회한다. 보통 위험하지만, body 클릭에는 안전—활성 모달 포커스 해제를 시도할 뿐이다.

## 더 넓은 패턴

Google 제품은 오버레이를 좋아한다:
- 키보드 단축키로 열리는 **검색 다이얼로그**
- 쿠키/추적용 **동의 프롬프트**
- 새 기능용 **온보딩 모달**
- 다른 Google 제품용 **프로모 배너**
- 평가 요청 **피드백 프롬프트**

이 오버레이들은 공통 특성을 공유:
- 전체 뷰포트 커버
- 모든 클릭 이벤트 캡처
- 시각적으로 투명하거나 미묘할 수 있음
- 에러 던지지 않음—입력만 훔침

## 예방 패턴

Google UI 자동화에는 방어적 오버레이 처리 추가:

```typescript
async function clickSafe(page: Page, selector: string): Promise<void> {
  await dismissOverlays(page);
  await page.waitForTimeout(200);  // 애니메이션 정착
  
  const element = page.locator(selector);
  await expect(element).toBeVisible({ timeout: 10000 });
  
  try {
    await element.click({ timeout: 5000 });
  } catch (error) {
    // 클릭 실패 시 force 클릭 폴백 시도
    console.warn(`Normal click failed, trying force click: ${selector}`);
    await element.click({ force: true });
  }
}
```

## 디버깅 체크리스트

Playwright 클릭이 보이는 엘리먼트에서 실패할 때:

1. **클릭 전 스크린샷** - 오버레이 찾기
2. **z-index 확인** - 더 높은 z-index 엘리먼트가 이벤트 캡처
3. **Escape 키 시도** - 잠재적 모달 닫기
4. **iframe 확인** - 엘리먼트가 프레임 안에 있을 수 있음
5. **force: true를 진단으로 사용** - force가 작동하면 뭔가 막고 있음

```typescript
// 진단: force가 작동하나?
await element.click({ force: true });  // 모든 체크 우회
// 이게 작동하고 일반 클릭이 안 되면 → 오버레이 문제
```

## 요약

버튼은 절대 망가지지 않았다. Google의 UI가 알 수 없는 이유로 거기 놓은 보이지 않는 레이어 뒤에 숨겨져 있었을 뿐이다.

Google 제품을 자동화할 때, 오버레이가 존재한다고 가정하라. 선제적으로 닫아라. 보이는 엘리먼트가 클릭 가능하다고 믿지 마라—볼 수 없는 뭔가가 가로막고 있을 수 있다.

"클릭 불가" 에러는 엘리먼트가 망가졌다는 뜻이 아니다. 다른 뭔가가 먼저 도착했다는 뜻이다.
