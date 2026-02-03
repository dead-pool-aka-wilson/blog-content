---
title: "라이트박스 라이브러리 버려라: 네이티브 Dialog가 더 낫다"
date: 2026-02-13
author: Raoul
categories: [technical]
tags: [html5, dialog, lightbox, no-js, accessibility]
draft: false
summary: "이미지 라이트박스에 12KB의 JavaScript를 배포하지 마라. 네이티브 HTML dialog 엘리먼트가 10줄로 더 나은 접근성과 함께 해낸다."
cover:
  image: "/covers/cover-2026-02-13-html5-dialog-lightbox.png"
---

이미지를 모달로 보여주는 것에 왜 12KB의 JavaScript를 배포하나?

청첩장 사이트에 갤러리를 만들고 있었다. 요구사항은 간단했다: 이미지 클릭, 더 큰 버전 보기, 클릭해서 닫기. 모든 튜토리얼이 Lightbox2, GLightbox, 또는 유사한 라이브러리를 가리켰다. 그 다음 `<dialog>` 엘리먼트가 존재한다는 걸 기억했다.

## WHY: 해결된 문제를 위한 라이브러리

이미지 라이트박스는 표준 동작이 있다:
- 트리거 시 열림
- 이미지 뒤에 어두운 백드롭
- Escape 키 또는 바깥 클릭으로 닫힘
- 뒤 콘텐츠와 상호작용 방지

이것이 정확히 네이티브 `<dialog>` 엘리먼트가 제공하는 것이다. 브라우저가 처리하는 것:
- 모달 백드롭 (어두운 오버레이)
- 포커스 트래핑 (Tab이 dialog 안에 머뭄)
- Escape 키로 닫기
- 접근성 속성

라이브러리는 이 모든 것을 JavaScript로 재구현하고, 엣지 케이스용 폴리필을 배포하고, 번들 무게를 추가한다. 네이티브 dialog는 그냥 작동한다.

## HOW: 구현

HTML은 최소다:

```html
<dialog id="lightbox" class="lightbox">
  <button class="lightbox-close" aria-label="Close">&times;</button>
  <img id="lightbox-image" src="" alt="" />
</dialog>
```

CSS가 외관을 처리한다:

```css
.lightbox {
  border: none;
  padding: 0;
  background: transparent;
  max-width: 95vw;
  max-height: 95vh;
}

.lightbox::backdrop {
  background: rgba(0, 0, 0, 0.95);
}

.lightbox img {
  max-width: 100%;
  max-height: 90vh;
  object-fit: contain;
}

.lightbox-close {
  position: absolute;
  top: -2rem;
  right: 0;
  background: none;
  border: none;
  color: white;
  font-size: 2rem;
  cursor: pointer;
}
```

JavaScript는 사소하다:

```javascript
const lightbox = document.getElementById('lightbox');
const lightboxImg = document.getElementById('lightbox-image');

// 이미지 클릭 시 열기
document.querySelectorAll('.gallery img').forEach(img => {
  img.addEventListener('click', () => {
    lightboxImg.src = img.src;
    lightboxImg.alt = img.alt;
    lightbox.showModal();
  });
});

// 버튼 클릭 시 닫기
document.querySelector('.lightbox-close').addEventListener('click', () => {
  lightbox.close();
});

// 백드롭 클릭 시 닫기
lightbox.addEventListener('click', (e) => {
  if (e.target === lightbox) lightbox.close();
});
```

끝이다. 10줄의 JavaScript, 의존성 제로, 완전한 기능.

## WHAT: 공짜로 얻는 것

### `showModal()` vs `show()`
`<dialog>` 엘리먼트는 두 가지 여는 메서드가 있다:

- `show()`: 비모달로 열림, 뒤 콘텐츠가 상호작용 가능
- `showModal()`: 백드롭과 함께 모달로 열림, 포커스 트랩, 상호작용 차단

라이트박스에는 `showModal()`이 원하는 것이다.

### `::backdrop` 의사 엘리먼트
모달 백드롭은 JavaScript로 생성된 div가 아닌 CSS 의사 엘리먼트다:

```css
dialog::backdrop {
  background: rgba(0, 0, 0, 0.95);
  backdrop-filter: blur(4px); /* 선택적 블러 효과 */
}
```

오버레이를 생성, 배치, 제거하는 JavaScript 필요 없음.

### 내장 포커스 관리
`showModal()`이 호출되면, 포커스가 dialog로 이동한다. Tab이 dialog 내에서 순환한다. 닫히면, 포커스가 트리거 엘리먼트로 돌아간다. 이 접근성 동작은 자동이다.

### Escape 키 처리
Escape를 누르면 `close` 이벤트가 발생하고 dialog가 닫힌다. 이벤트 리스너 필요 없음—기본 동작이다.

### 브라우저 지원
`<dialog>`는 모든 현대 브라우저(Chrome, Firefox, Safari, Edge)에서 95%+ 글로벌 커버리지로 지원된다. IE11을 타겟하지 않는 한 폴리필 필요 없음.

## 크기 비교

| 라이브러리 | 크기 | 의존성 |
|------------|------|--------|
| Lightbox2 | 8KB + jQuery | jQuery 필요 |
| GLightbox | 12KB | 없음 |
| PhotoSwipe | 45KB | 없음 |
| 네이티브 `<dialog>` | 0KB | 없음 |

네이티브로 존재하는 브라우저 기능을 재구현하기 위해 킬로바이트의 JavaScript를 배포하고 있다.

### 여전히 라이브러리가 필요할 때

라이브러리는 기본 라이트박스 이상의 기능을 제공한다:
- 이미지 줌과 팬
- 썸네일이 있는 슬라이드쇼
- 모바일에서 스와이프 제스처
- 지연 로딩 갤러리

이것들이 필요하면, 라이브러리가 말이 된다. 하지만 "클릭해서 확대, 클릭해서 닫기"—가장 일반적인 사용 사례—에는 네이티브 dialog가 충분하다.

### 점진적 향상

라이트박스는 우아하게 저하된다. JavaScript가 실패하면:
- 이미지는 여전히 보임
- 클릭해도 아무것도 안 함 (망가진 게 아니라, 그냥 향상되지 않음)

JavaScript 에러 없음, 망가진 레이아웃 없음. 갤러리는 라이트박스 기능 없이도 작동한다.

## 요약

"라이트박스는 그렇게 하는 거니까" GLightbox를 청첩장에 거의 추가할 뻔했다. 그 다음 물었다: 이미 존재하는 브라우저 기능이 뭔가?

`<dialog>` 엘리먼트가 문제를 완전히 해결했다:
- 의존성 제로
- 자동 접근성
- 브라우저 처리 escape와 포커스
- 10줄의 JavaScript

플랫폼이 제공하는 것을 확인하기 전에 라이브러리에 손 뻗지 마라. HTML은 진화했다. 스택 트레이스는 그럴 필요 없다.
