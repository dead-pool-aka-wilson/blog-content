---
title: "Recreating Wong Kar-wai's Film Aesthetic with Cloudinary"
date: 2026-02-21
author: Raoul
categories: [ai-prompting]
tags: [cloudinary, image-processing, css, film-aesthetic]
draft: false
summary: "No Photoshop. No local image editing. Just CDN transforms that turn stock photos into moody Hong Kong cinema."
---

I wanted a Korean wedding invitation that felt like a Wong Kar-wai film. Deep reds, desaturated tones, that melancholic warmth of *In the Mood for Love*. The photos were generic stock images. They needed to feel like 1960s Hong Kong.

The solution wasn't Photoshop. It was Cloudinary URL parameters.

## The Prompt

> Apply Wong Kar-wai cinematic color grading to photos using Cloudinary transforms.

With a code example:

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

## Why This Worked

### 1. Reference Point Was Specific

"Wong Kar-wai" isn't a vague aesthetic request. It's a specific visual language:
- Saturated reds and greens
- High contrast shadows
- Warm, amber-tinted highlights
- Desaturated midtones
- Film grain texture

Anyone who's seen *Chungking Express* or *2046* knows this look. The reference did the specification work.

### 2. Implementation Path Was Clear

Cloudinary effects map directly to color grading:
- `colorize` → color overlay
- `saturation` → saturation adjustment
- `contrast` → contrast curve

The prompt connected the aesthetic goal to the implementation tool.

### 3. Code Example Seeded the Pattern

Starting with a working example, even partial, gives the AI something to build on. The example showed:
- Cloudinary's effect syntax
- The general approach (layer multiple effects)
- That code-level integration was expected

## The Implementation

Different layers got different treatments to create depth:

```javascript
const wkwGrade = {
  // Background: heavy red, dreamy
  background: [
    { colorize: '20,rgb:8B0000' },  // 20% deep red overlay
    { saturation: -10 },
    { contrast: 15 }
  ],
  
  // Midground: moody green, cinematic
  midground: [
    { colorize: '15,rgb:2D4A3E' },  // forest green
    { saturation: 10 },
    { contrast: 20 }
  ],
  
  // Foreground: warm amber, intimate
  foreground: [
    { colorize: '10,rgb:D2691E' },  // amber/chocolate
    { saturation: 5 },
    { contrast: 25 }
  ]
};
```

Each layer gets progressively less colorization and more contrast, creating a focal depth effect.

### Why CDN Processing Matters

This approach has advantages over local image editing:

| Aspect | Photoshop | Cloudinary |
|--------|-----------|------------|
| Source files | Must edit each | Use originals |
| Responsive sizes | Manual export | Automatic |
| Consistency | Manual matching | Code-defined |
| Changes | Re-edit all | Update params |
| Performance | Serve edited files | CDN cached |

The color grading is defined in code. Change the parameters, and every image updates instantly.

### The Color Choices

Wong Kar-wai's cinematographer Christopher Doyle used specific color palettes:

**Deep Red** (`rgb:8B0000`):
- Dark, romantic, intimate
- Think: *In the Mood for Love* stairwell scenes
- Used for backgrounds to create warmth

**Moody Green** (`rgb:2D4A3E`):
- Urban, lonely, neon-adjacent
- Think: *Chungking Express* convenience store
- Used for midground depth

**Warm Amber** (`rgb:D2691E`):
- Golden hour, nostalgic
- Think: *2046* hotel corridors
- Used for foreground intimacy

## Usage Pattern

For any image that needs the treatment:

```astro
<CldImage
  src={imageId}
  effects={wkwGrade.background}
  width={1920}
  height={1080}
  alt="Hong Kong skyline with WKW grading"
/>
```

The effect array is applied server-side at Cloudinary's CDN. The client receives a pre-graded image at the requested dimensions.

## When This Approach Works

CDN-side color grading makes sense when:
- You have many images needing consistent treatment
- Responsive sizing is required
- The effect is purely color/contrast (not masking or compositing)
- You want to change the look without re-processing files

It doesn't work for:
- Complex retouching (skin, blemishes)
- Layer masking or selective adjustments
- Effects requiring image-specific tuning

## The Takeaway

The prompt worked because it connected a specific aesthetic reference (Wong Kar-wai) to a specific implementation tool (Cloudinary effects) with a working code example.

Generic stock photos now feel like moody Hong Kong cinema. No local image editing, no asset pipeline complexity. Just URL parameters that turn `colorize: '20,rgb:8B0000'` into nostalgia.

Sometimes the best Photoshop alternative is not Photoshop at all.
