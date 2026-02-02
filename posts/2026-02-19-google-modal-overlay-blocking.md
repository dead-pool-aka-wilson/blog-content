---
title: "The Invisible Modal: Debugging Playwright Clicks on Google UIs"
date: 2026-02-19
author: Raoul
categories: [debugging]
tags: [playwright, google-ui, automation, modal]
draft: false
summary: "The button was visible. The selector found it. The click did nothing. An invisible modal overlay was stealing every interaction."
---

"Add source button not clickable."

I could see the button in the screenshot. Playwright found the element—the selector worked. But `click()` either failed silently or timed out. The button existed. It just wouldn't respond.

## The Symptom

Automating NotebookLM uploads with Playwright:

```typescript
// Find the button - succeeds
const addButton = await page.locator('button[aria-label="Add source"]');
await expect(addButton).toBeVisible();  // Passes!

// Click the button - fails
await addButton.click();  // Timeout or silent failure
```

The element was found. Visibility checks passed. Clicks didn't work.

## The Investigation

Playwright's `click()` does more than trigger a click event. It:
1. Scrolls the element into view
2. Waits for the element to be stable
3. Checks that the element is actionable (visible, enabled, not obscured)
4. Clicks the center of the element

Step 4 can fail if something is covering the element—even if that something is invisible.

Adding a debug screenshot before the click revealed the problem:

```typescript
await page.screenshot({ path: 'debug-before-click.png' });
```

The screenshot showed a semi-transparent overlay covering the entire page. Google had opened a search dialog, or a consent prompt, or a promo modal. The overlay was visually subtle but captured all click events.

## The Fix

Created a `dismissOverlays()` function to clear modals before interacting:

```typescript
async function dismissOverlays(page: Page): Promise<void> {
  // Press Escape multiple times to dismiss any open dialogs
  for (let i = 0; i < 3; i++) {
    await page.keyboard.press("Escape");
    await page.waitForTimeout(200);
  }
  
  // Click outside any modal areas (top-left corner)
  try {
    await page.click("body", { 
      position: { x: 0, y: 0 }, 
      force: true 
    });
  } catch {
    // Ignore if click fails - body might not be clickable
  }
  
  // Wait for any animations to settle
  await page.waitForTimeout(300);
}
```

Usage before critical interactions:

```typescript
await dismissOverlays(page);
await page.locator('button[aria-label="Add source"]').click();
```

### Why Multiple Escapes?

Google UIs can have nested modals:
- Search overlay
- Consent dialog underneath
- Onboarding tooltip underneath that

One Escape might close the top layer, revealing another. Three Escapes handles reasonable nesting.

### Why `force: true`?

The `force: true` option bypasses Playwright's actionability checks. Normally risky, but safe for the body click—we're just trying to unfocus any active modal.

## The Broader Pattern

Google products love overlays:
- **Search dialogs** that open on keyboard shortcuts
- **Consent prompts** for cookies/tracking
- **Onboarding modals** for new features
- **Promo banners** for other Google products
- **Feedback prompts** asking for ratings

These overlays share common traits:
- Cover the full viewport
- Capture all click events
- May be visually transparent or subtle
- Don't throw errors—just steal input

## Prevention Pattern

For any Google UI automation, add defensive overlay handling:

```typescript
async function clickSafe(page: Page, selector: string): Promise<void> {
  await dismissOverlays(page);
  await page.waitForTimeout(200);  // Let animations settle
  
  const element = page.locator(selector);
  await expect(element).toBeVisible({ timeout: 10000 });
  
  try {
    await element.click({ timeout: 5000 });
  } catch (error) {
    // If click fails, try force click as fallback
    console.warn(`Normal click failed, trying force click: ${selector}`);
    await element.click({ force: true });
  }
}
```

## Debugging Checklist

When Playwright clicks fail on visible elements:

1. **Screenshot before clicking** - Look for overlays
2. **Check z-index** - Higher z-index elements capture events
3. **Try Escape key** - Dismiss potential modals
4. **Check for iframes** - Element might be in a frame
5. **Use force: true as diagnostic** - If force works, something's blocking

```typescript
// Diagnostic: does force work?
await element.click({ force: true });  // Bypasses all checks
// If this works but normal click doesn't → overlay issue
```

## The Takeaway

The button was never broken. It was just hidden behind an invisible layer that Google's UI placed there for reasons unknown.

When automating Google products, assume overlays exist. Dismiss them proactively. Don't trust that a visible element is clickable—something you can't see might be in the way.

The error "not clickable" doesn't mean the element is broken. It means something else got there first.
