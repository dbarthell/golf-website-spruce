---
id: GOL-v14whv
type: task
title: Logo enhancement
status: Done
---
# Logo enhancement

Would you be able to make the flag yellow in the logo?

> **Update:** After review, the flag was made red with a yellow flagstick instead. The new logo (`images/red-flag-logo.jpg`) was created externally and wired into the app.

---

## Implementation Plan

### Summary

The correct logo is `images/new-logo.jpg` — a dark navy square showing the "ZB" monogram with a golf flagstick and flag integrated into the "B". The flag is currently the same white/cream color as the rest of the letterforms. The task is to make the flag **Masters yellow** — the same golden yellow as the flags at Augusta National.

Two issues need to be addressed together:

1. **The file is deleted** — `new-logo.jpg` was removed from the working tree (still exists in git history)
2. **The wrong logo is wired up** — [`src/pages/LookupPage.tsx`](code://src/pages/LookupPage.tsx) line 167 still references `images/logo.png` (the old wavy-line logo), not `new-logo.jpg`
3. **The flag needs to be yellow** — the flag shape in the logo must be recolored

---

### The Logo

`images/new-logo.jpg` (recoverable from git): a navy blue square badge with "ZB" letterforms; the vertical stroke of the "B" doubles as a flagstick with a triangular flag pointing right — currently white. This needs to become **Masters yellow** — Augusta National flag color, approximately `#FFC72C`.

---

### The Challenge: Raster vs. Vector

`new-logo.jpg` is a **JPEG raster image** — individual pixels, no editable shape layers. This means the flag color **cannot be changed with CSS alone**. Options:

**Option A — Recreate as SVG (Recommended)**
Convert the logo to SVG format so the flag shape gets its own `<path>` with a `fill` attribute. Benefits:

- Flag color (and any other color) can be changed trivially at any time via CSS or SVG attribute
- Crisp at all sizes (no pixelation at small `2rem × 2rem` header size)
- Can be inlined in React or kept as a `.svg` file

Steps:

1. Recreate `images/logo.svg` (traced from the JPG, or rebuilt from the original design source)
2. Assign Masters yellow fill (`#FFC72C`) to the flag path, keep the rest white/cream
3. Update [`src/pages/LookupPage.tsx`](code://src/pages/LookupPage.tsx) line 167: change `images/logo.png` → `images/logo.svg`
4. Confirm visual fit with `.header-logo { width: 2rem; height: 2rem; border-radius: 6px; }` in [`src/styles/global.css`](code://src/styles/global.css)

**Option B — Edit the JPEG in an image editor**
Open in Photoshop, Figma, or similar; select the flag pixels; recolor to yellow; export a new `new-logo.jpg` (or `logo.png`).

Steps:

1. Restore `new-logo.jpg` from git: `git checkout HEAD -- images/new-logo.jpg`
2. Edit the flag pixels to Masters yellow (`#FFC72C`) in an image editor
3. Save the result back to `images/new-logo.jpg`
4. Update [`src/pages/LookupPage.tsx`](code://src/pages/LookupPage.tsx) line 167: change `images/logo.png` → `images/new-logo.jpg`

---

### Open Question (Needs Human Input)

> **Do you have the original source file** for the new logo (e.g. from the AI image generator, Figma, Illustrator)? If so, regenerating/exporting with a yellow flag would be the fastest path. Otherwise, the SVG recreation or pixel-editing approach above can be used.

---

### Files to Change

| File | Change |
| --- | --- |
| `images/new-logo.jpg` (or new `logo.svg`) | Restore from git + recolor flag to yellow, or create SVG version |
| [`src/pages/LookupPage.tsx`](code://src/pages/LookupPage.tsx) line 167 | Change `src="images/logo.png"` → correct filename |

No other files need to change.