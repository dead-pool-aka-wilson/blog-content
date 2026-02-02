---
date: 2026-02-02
session: ses_3e5f55063ffeeNTGeDLcZfesh1
tags: [html5, dialog, lightbox, no-js, accessibility]
stage: seed
content_potential: medium
---

## The Idea

Use native HTML `<dialog>` element for image lightbox instead of JavaScript libraries.

## Context

Building a gallery lightbox for a wedding invitation. The plan said "NO complex JavaScript lightbox libraries" to keep dependencies minimal.

## Raw Exchange

> Plan: "Implement CSS-only lightbox using native `<dialog>`"
> Implementation: 
```html
<dialog id="lightbox" class="lightbox">
  <button class="lightbox-close">&times;</button>
  <img id="lightbox-image" />
</dialog>
```

```javascript
lightbox.showModal();  // opens with backdrop
lightbox.close();      // closes
```

## Expansion Potential

**Article**: "Ditch Your Lightbox Library: Native Dialog Does It Better"

Key points:
1. `showModal()` vs `show()` - modal blocks interaction, non-modal doesn't
2. `::backdrop` pseudo-element for overlay styling
3. Escape key closes automatically
4. Click outside closes with simple event listener
5. Focus trap is built-in
6. Works in all modern browsers (95%+ support)

**Code comparison**:
- Lightbox2: 8KB + jQuery dependency
- GLightbox: 12KB
- Native dialog: 0KB, 10 lines of JS

## Technical Notes

```css
dialog::backdrop {
  background: rgba(0, 0, 0, 0.95);
}

dialog {
  border: none;
  max-width: 100vw;
  max-height: 100vh;
}
```
