---
date: 2026-02-02
session: ses_3e5f55063ffeeNTGeDLcZfesh1
tags: [lenis, scroll-snap, css, javascript, gotcha]
stage: seed
content_potential: high
---

## The Idea

Lenis smooth scroll library is INCOMPATIBLE with CSS scroll-snap. They fight for scroll control.

## Context

Wedding invitation site had both Lenis (for smooth parallax scrolling) and CSS scroll-snap (for section-by-section navigation). Neither worked properly.

## Raw Exchange

> Librarian agent research: "CRITICAL: Lenis and CSS scroll-snap are INCOMPATIBLE - Lenis hijacks native scrolling"
> Solution: "Remove Lenis, use CSS scroll-snap + native smooth scroll for better mobile performance"

## Investigation

Lenis works by:
1. Intercepting wheel/touch events
2. Applying its own scroll animation via `requestAnimationFrame`
3. Updating scroll position programmatically

CSS scroll-snap expects:
1. Native scroll behavior
2. Browser to handle snap points
3. No interference with scroll position

When combined:
- Lenis animates scroll → browser tries to snap → Lenis overrides → janky behavior
- Mobile especially broken (touch events fight)

## Resolution

Removed Lenis entirely:
```javascript
// Before (Layout.astro)
import Lenis from 'lenis';
const lenis = new Lenis({ ... });
lenis.on('scroll', ScrollTrigger.update);

// After
// Just GSAP ScrollTrigger, no Lenis
gsap.registerPlugin(ScrollTrigger);
```

Added native smooth scroll:
```css
html {
  scroll-behavior: smooth;
  scroll-snap-type: y proximity;
}

section {
  scroll-snap-align: start;
}
```

## Lesson

Don't mix scroll-hijacking libraries with CSS scroll-snap. Choose one:
- **Lenis/Locomotive**: Complex custom scroll experiences, no snap
- **CSS scroll-snap**: Section-based navigation, simpler, better mobile performance

## Expansion Potential

**Article**: "Why Lenis Broke My Scroll-Snap (And How I Fixed It)"
- Common gotcha for developers combining smooth scroll libs with snap
- Debugging steps: check if library uses `scrollTo()` or position manipulation
- Alternative: GSAP ScrollTrigger `snap` property (works with native scroll)
