---
title: "Why Lenis Broke My Scroll-Snap (And How I Fixed It)"
date: 2026-02-08
author: Raoul
categories: [technical]
tags: [lenis, scroll-snap, css, javascript, gotcha]
draft: false
summary: "Lenis smooth scroll and CSS scroll-snap are fundamentally incompatible. They both want control of scroll position, and only one can win."
---

The scroll behavior was broken in a way I couldn't explain.

I was building a wedding invitation site with full-page sections. Scroll to a section, it snaps into place. Simple enough—CSS `scroll-snap` has been reliable for years. But I also wanted that buttery-smooth scroll feel, so I added Lenis.

The result? Janky half-snaps, weird momentum, and mobile completely broken. Two features that work perfectly alone, fighting for control when combined.

## WHY: Two Scroll Controllers, One Scroll

Lenis and CSS scroll-snap both assume they're in charge of scroll behavior. That's the fundamental conflict.

**How Lenis works:**
1. Intercepts wheel and touch events
2. Calculates its own smooth animation
3. Updates `scrollTop` programmatically via `requestAnimationFrame`
4. Provides that smooth, slightly-heavy scroll feel

**How CSS scroll-snap works:**
1. Observes native scroll position
2. When scroll ends near a snap point, browser takes over
3. Browser animates to the nearest snap point
4. Relies on untampered native scroll behavior

When you combine them:
1. You scroll → Lenis intercepts and animates
2. Lenis updates scroll position → Browser detects a stop
3. Browser tries to snap → Lenis is still animating → Fight
4. Result: stuttering, double-animations, or no snap at all

On mobile, it's worse. Touch events are more complex, and Lenis's interception conflicts with both snap behavior and native momentum scrolling.

## HOW: Debugging the Conflict

The symptoms were confusing at first. Sometimes sections snapped; sometimes they didn't. Scroll felt "sticky" in a bad way. I initially blamed scroll-snap configuration and wasted an hour tweaking `scroll-snap-type` values.

The breakthrough came from a librarian agent I use for research. It surfaced the critical insight:

> "Lenis and CSS scroll-snap are INCOMPATIBLE - Lenis hijacks native scrolling"

Once I understood the mechanism, the fix was obvious: remove one of them.

### Diagnosing Your Own Scroll Conflicts

If you're seeing similar symptoms, check whether your scroll library:

1. **Uses `scrollTo()` or direct position manipulation** - This conflicts with browser snap behavior
2. **Intercepts wheel/touch events** - Prevents native scroll from reaching the browser
3. **Runs its own animation loop** - Competes with browser's snap animation

Libraries that hijack scroll: Lenis, Locomotive Scroll, smooth-scrollbar
Libraries that enhance without hijacking: GSAP ScrollTrigger (when not using smoothScroll)

## WHAT: The Resolution

I removed Lenis entirely and relied on native CSS:

**Before (broken):**
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

**After (working):**
```javascript
// Layout.astro
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);
// No Lenis - let native scroll handle it
```

```css
/* Native smooth scroll + snap */
html {
  scroll-behavior: smooth;
  scroll-snap-type: y proximity;
}

section {
  scroll-snap-align: start;
  scroll-snap-stop: normal;
}
```

The `scroll-behavior: smooth` CSS property gives you smooth scrolling for anchor links without JavaScript. Combined with `scroll-snap-type: y proximity`, you get sections that snap when you're close but don't force snapping on every scroll.

### If You Need Both Effects

Sometimes you genuinely want custom scroll physics AND snap points. Options:

**GSAP ScrollTrigger snap** - Works with native scroll:
```javascript
ScrollTrigger.create({
  snap: {
    snapTo: 1 / (sections.length - 1),
    duration: { min: 0.2, max: 0.3 },
    ease: "power1.inOut"
  }
});
```

This snaps using GSAP's animation system but doesn't hijack the scroll itself. It observes scroll position and applies snap animation when appropriate.

**Lenis with manual snap points** - If you must use Lenis, disable CSS snap and implement your own snap logic in the `scroll` callback. More work, but possible.

**Accept the tradeoff** - On most modern browsers, native smooth scroll is good enough. The difference between native and Lenis is noticeable but not dramatic. Unless you're building a portfolio site where scroll feel is the product, native wins on reliability.

### Mobile Considerations

CSS scroll-snap has excellent mobile support. Lenis on mobile is hit-or-miss—touch event handling varies by device and browser, and the "smooth" effect often feels like input lag rather than polish.

For the wedding site, removing Lenis actually improved mobile experience. Native scroll + snap felt more responsive than the Lenis-flavored alternative.

## The Lesson

Don't mix scroll-hijacking libraries with CSS scroll-snap. They're solving the same problem (controlling scroll behavior) with incompatible approaches.

Choose based on your needs:
- **CSS scroll-snap**: Section-based navigation, better mobile performance, less JavaScript
- **Lenis/Locomotive**: Highly customized scroll physics, no snap, JavaScript-heavy

If you need both smooth scroll and snapping, use GSAP ScrollTrigger's snap feature or implement custom snap logic. Don't rely on two systems fighting for the same input.

The hour I spent debugging a "CSS scroll-snap bug" was actually a architectural conflict I created by combining incompatible tools. Sometimes the fix isn't tweaking configuration—it's removing a dependency.
