---
title: "Lenis가 내 Scroll-Snap을 망친 이유 (그리고 해결 방법)"
date: 2026-02-08
author: Raoul
categories: [technical]
tags: [lenis, scroll-snap, css, javascript, gotcha]
draft: false
summary: "Lenis 스무스 스크롤과 CSS scroll-snap은 근본적으로 호환되지 않는다. 둘 다 스크롤 위치를 제어하려 하고, 하나만 이길 수 있다."
cover:
  image: "/covers/cover-2026-02-08-lenis-scrollsnap-incompatibility.png"
---

스크롤 동작이 설명할 수 없는 방식으로 망가졌다.

풀페이지 섹션이 있는 청첩장 사이트를 만들고 있었다. 섹션으로 스크롤하면 제자리에 스냅된다. 간단하다—CSS `scroll-snap`은 몇 년간 신뢰할 수 있었다. 하지만 버터처럼 부드러운 스크롤 느낌도 원했기에 Lenis를 추가했다.

결과? 버벅거리는 반쪽 스냅, 이상한 모멘텀, 모바일은 완전히 망가짐. 각각 완벽하게 작동하는 두 기능이, 합쳐지면 제어권을 두고 싸운다.

## WHY: 두 개의 스크롤 컨트롤러, 하나의 스크롤

Lenis와 CSS scroll-snap은 둘 다 자기가 스크롤 동작을 담당한다고 가정한다. 그게 근본적인 충돌이다.

**Lenis 작동 방식:**
1. 휠과 터치 이벤트 가로채기
2. 자체 스무스 애니메이션 계산
3. `requestAnimationFrame`을 통해 프로그래밍 방식으로 `scrollTop` 업데이트
4. 부드럽고, 약간 무거운 스크롤 느낌 제공

**CSS scroll-snap 작동 방식:**
1. 네이티브 스크롤 위치 관찰
2. 스냅 포인트 근처에서 스크롤이 끝나면, 브라우저가 인계
3. 브라우저가 가장 가까운 스냅 포인트로 애니메이션
4. 변조되지 않은 네이티브 스크롤 동작에 의존

합치면:
1. 스크롤 → Lenis가 가로채고 애니메이션
2. Lenis가 스크롤 위치 업데이트 → 브라우저가 멈춤 감지
3. 브라우저가 스냅 시도 → Lenis는 아직 애니메이션 중 → 충돌
4. 결과: 버벅거림, 이중 애니메이션, 또는 스냅 없음

모바일에서는 더 심하다. 터치 이벤트가 더 복잡하고, Lenis의 가로채기가 스냅 동작과 네이티브 모멘텀 스크롤링 둘 다와 충돌한다.

## HOW: 충돌 디버깅

처음에 증상이 혼란스러웠다. 섹션이 스냅될 때도 있고 안 될 때도 있었다. 스크롤이 나쁜 방식으로 "끈적"했다. 처음에는 scroll-snap 설정 탓을 하고 `scroll-snap-type` 값을 조정하느라 한 시간을 낭비했다.

돌파구는 연구에 사용하는 라이브러리안 에이전트에서 왔다. 중요한 통찰을 찾아냈다:

> "Lenis와 CSS scroll-snap은 호환되지 않는다 - Lenis가 네이티브 스크롤링을 하이재킹한다"

메커니즘을 이해하고 나니, 해결책은 분명했다: 둘 중 하나를 제거한다.

### 자신의 스크롤 충돌 진단하기

비슷한 증상이 보인다면, 스크롤 라이브러리가 다음을 하는지 확인하라:

1. **`scrollTo()` 또는 직접 위치 조작 사용** - 브라우저 스냅 동작과 충돌
2. **휠/터치 이벤트 가로채기** - 네이티브 스크롤이 브라우저에 도달 못함
3. **자체 애니메이션 루프 실행** - 브라우저의 스냅 애니메이션과 경쟁

스크롤을 하이재킹하는 라이브러리: Lenis, Locomotive Scroll, smooth-scrollbar
하이재킹 없이 향상하는 라이브러리: GSAP ScrollTrigger (smoothScroll을 사용하지 않을 때)

## WHAT: 해결책

Lenis를 완전히 제거하고 네이티브 CSS에 의존했다:

**이전 (망가짐):**
```javascript
// Layout.astro
import Lenis from 'lenis';

const lenis = new Lenis({
  duration: 1.2,
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
});

lenis.on('scroll', ScrollTrigger.update);

function raf(time) {
  lenis.raf(time);
  requestAnimationFrame(raf);
}
requestAnimationFrame(raf);
```

**이후 (작동함):**
```javascript
// Layout.astro
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);
// No Lenis - 네이티브 스크롤이 처리하게
```

```css
/* 네이티브 스무스 스크롤 + 스냅 */
html {
  scroll-behavior: smooth;
  scroll-snap-type: y proximity;
}

section {
  scroll-snap-align: start;
  scroll-snap-stop: normal;
}
```

`scroll-behavior: smooth` CSS 속성은 JavaScript 없이 앵커 링크에 스무스 스크롤을 제공한다. `scroll-snap-type: y proximity`와 합치면, 가까울 때 스냅하지만 모든 스크롤에 스냅을 강제하지 않는 섹션을 얻는다.

### 두 효과가 모두 필요할 때

정말로 커스텀 스크롤 물리학과 스냅 포인트가 둘 다 필요할 때가 있다. 옵션:

**GSAP ScrollTrigger snap** - 네이티브 스크롤과 작동:
```javascript
ScrollTrigger.create({
  snap: {
    snapTo: 1 / (sections.length - 1),
    duration: { min: 0.2, max: 0.3 },
    ease: "power1.inOut"
  }
});
```

GSAP의 애니메이션 시스템을 사용해 스냅하지만 스크롤 자체를 하이재킹하지 않는다. 스크롤 위치를 관찰하고 적절할 때 스냅 애니메이션을 적용한다.

**수동 스냅 포인트가 있는 Lenis** - Lenis를 꼭 사용해야 한다면, CSS 스냅을 비활성화하고 `scroll` 콜백에서 자체 스냅 로직을 구현하라. 더 많은 작업이지만, 가능하다.

**트레이드오프 수용** - 대부분의 현대 브라우저에서, 네이티브 스무스 스크롤은 충분히 좋다. 네이티브와 Lenis의 차이는 눈에 띄지만 극적이지 않다. 스크롤 느낌이 제품인 포트폴리오 사이트를 만드는 게 아니라면, 신뢰성에서 네이티브가 이긴다.

### 모바일 고려사항

CSS scroll-snap은 훌륭한 모바일 지원을 가지고 있다. 모바일의 Lenis는 들쭉날쭉—터치 이벤트 처리가 기기와 브라우저에 따라 다르고, "스무스" 효과가 종종 폴리시가 아니라 입력 지연처럼 느껴진다.

청첩장 사이트의 경우, Lenis 제거가 실제로 모바일 경험을 개선했다. 네이티브 스크롤 + 스냅이 Lenis 맛 대안보다 더 반응적으로 느껴졌다.

## 교훈

스크롤 하이재킹 라이브러리와 CSS scroll-snap을 섞지 마라. 둘 다 같은 문제(스크롤 동작 제어)를 호환되지 않는 접근법으로 해결한다.

필요에 따라 선택하라:
- **CSS scroll-snap**: 섹션 기반 내비게이션, 더 나은 모바일 성능, 적은 JavaScript
- **Lenis/Locomotive**: 고도로 커스터마이즈된 스크롤 물리학, 스냅 없음, JavaScript 헤비

스무스 스크롤과 스냅이 둘 다 필요하면, GSAP ScrollTrigger의 스냅 기능을 사용하거나 커스텀 스냅 로직을 구현하라. 같은 입력을 두고 싸우는 두 시스템에 의존하지 마라.

"CSS scroll-snap 버그"를 디버깅하느라 쓴 한 시간은 실제로 호환되지 않는 도구를 결합해서 내가 만든 아키텍처 충돌이었다. 때로는 수정이 설정 조정이 아니라—의존성 제거다.
