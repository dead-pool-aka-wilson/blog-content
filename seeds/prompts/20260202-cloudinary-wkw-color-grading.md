---
date: 2026-02-02
project: dead-pool-aka-wilson.github.io
session: ses_3e5f55063ffeeNTGeDLcZfesh1
tags: [cloudinary, image-processing, css, film-aesthetic]
content_potential: high
---

## The Prompt

Apply Wong Kar-wai cinematic color grading to photos using Cloudinary transforms:

```astro
<CldImage
  src="wedding/parallax/hk-skyline"
  effects={[
    { colorize: '20,rgb:8B0000' },  // deep red overlay
    { saturation: -10 },             // desaturated
    { contrast: 15 }                 // higher contrast
  ]}
/>
```

## Context

Building a Korean wedding invitation with "Hong Kong Night" aesthetic inspired by Wong Kar-wai films (In the Mood for Love, Chungking Express). Needed to apply consistent film-like color grading to stock photos.

## Outcome

Worked perfectly. Different colorize values per layer created depth:
- Background layer: `colorize: '20,rgb:8B0000'` (heavy red)
- Mid layer: `colorize: '15,rgb:2D4A3E'` (moody green)
- Foreground: `colorize: '10,rgb:D2691E'` (warm amber)

## Notable Because

1. **CDN-side processing**: No local image editing needed
2. **Responsive**: Cloudinary handles different sizes automatically
3. **Consistent aesthetic**: Same effect applied across all photos
4. **Performance**: Cached at CDN edge, no client-side processing

## Code Pattern

```javascript
// WKW color palette as Cloudinary effects
const wkwGrade = {
  background: [{ colorize: '20,rgb:8B0000' }, { saturation: -10 }, { contrast: 15 }],
  midground: [{ colorize: '15,rgb:2D4A3E' }, { saturation: 10 }, { contrast: 20 }],
  foreground: [{ colorize: '10,rgb:D2691E' }, { saturation: 5 }, { contrast: 25 }],
};
```

## Potential Article

"Recreating Wong Kar-wai's Film Aesthetic with Cloudinary Transforms"
- Target: Frontend developers interested in cinematic web design
- Angle: How to achieve film-like color grading without Photoshop
