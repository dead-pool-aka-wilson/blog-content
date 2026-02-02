---
date: 2026-02-01
project: moltbot/notebook-lm skill
session: ses_3e5ce2650ffezITFBkVK17PzIN
severity: moderate
resolved: true
tags: [playwright, google-ui, automation, modal]
---

## What Happened

Playwright automation for NotebookLM upload failed with "Add source button not clickable". Screenshots showed the button was visible but clicks didn't register.

## Investigation

Screenshot analysis revealed: Google's UI had an invisible modal overlay (search dialog, promo, or consent dialog) capturing all clicks before they reached the target button.

Key symptom: Element is visible, selector finds it, but `click()` fails silently or times out.

## Resolution

Created `dismissOverlays()` function to clear modals before interacting:

```typescript
async function dismissOverlays(page: Page) {
  // Press Escape multiple times to dismiss any open dialogs
  for (let i = 0; i < 3; i++) {
    await page.keyboard.press("Escape");
    await page.waitForTimeout(200);
  }
  
  // Click outside any modal areas
  try {
    await page.click("body", { position: { x: 0, y: 0 }, force: true });
  } catch {
    // Ignore if click fails
  }
}

// Usage before any critical interaction
await dismissOverlays(page);
await page.click('button[aria-label="Add source"]');
```

## Lesson

**Google products love modals.** When automating Google UIs (Docs, Drive, NotebookLM, etc.), always:

1. Take debug screenshots before failing
2. Assume invisible overlays exist
3. Add `dismissOverlays()` as standard practice
4. Use `force: true` as last resort (bypasses actionability checks)

## Prevention Pattern

```typescript
// Before any critical Google UI interaction:
await dismissOverlays(page);
await page.waitForTimeout(500); // Let animations settle
await page.click(selector, { timeout: 10000 });
```
