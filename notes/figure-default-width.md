# Debug Note: Information Section Losing Center Alignment

## Issue
`#information-section` (a flex container with `justify-content: center` and
`align-items: center`) stopped visually centering its content once
`.template-article`'s `max-width` was reduced to `300px` or lower.

## Root Cause

### 1. Unreset `<figure>` default margin
Browsers apply a default UA stylesheet rule:

```css
figure {
  display: block;
  margin: 1em 40px; /* 40px left + 40px right by default */
}
```

This was never reset in the stylesheet, so every `<figure>` inside
`.template-article` carried an invisible **80px of horizontal margin**
(40px each side) that wasn't accounted for in the layout.

### 2. Fixed-width caption + capped-width image
- `.info-image` → `max-width: 250px` (image renders up to 250px, does not
  shrink below that since no `width: 100%` was set)
- `.image-caption` → `width: 250px` (hardcoded, not `max-width`, so it
  never shrinks with its parent)

The `<figure>`'s content wants ~250px, but its margin box needs
`250px + 80px = 330px`. At `max-width: 320px` this was already tight but
easy to miss. At `300px` or below, the overflow became pronounced — the
image/caption don't shrink to fit, so they visually overflow their flex
item's box.

**Effect:** `justify-content`/`align-items: center` were still centering
the boxes correctly *as flexbox understood them*. The visual misalignment
came from actual rendered content overflowing outside those boxes, not
from a flexbox centering bug.

### 3. Secondary compounding issue: no `box-sizing: border-box` reset
`.info-image` has `max-width: 250px` plus a `2px` border. With the default
`content-box` sizing, the border is added on top, making the true
rendered width `254px` instead of `250px`. Minor alone, but it stacks with
the issue above and makes manual box-size math unreliable.

## Fix

```css
/* Reset default figure margin */
.template-article figure {
  margin: 0;
}

/* Make caption fluid instead of fixed-width */
.image-caption {
  width: 100%;      /* was: width: 250px */
  max-width: 250px;
  box-sizing: border-box;
}
```

## Lesson
Flexbox centering (`justify-content`/`align-items`) is only as accurate as
the box sizes it's given. If a descendant element overflows its flex item
silently (via unreset default margins, fixed widths, or unaccounted
borders), no amount of centering CSS on the parent will fix the visual
result — because the problem isn't the centering logic, it's that the
boxes being centered don't match what's actually rendering.

**Takeaway habit:** always reset `<figure>`/`<blockquote>` default
margins, prefer `max-width` over fixed `width` for anything meant to be
responsive, and apply a global `box-sizing: border-box` reset early in
the stylesheet.
