---
id: GOL-6ry6v4
type: feature
title: New logo
status: In Review
---
# New logo

I added a new logo (new-logo jpg) to the images folder in the repo. let’s replace that with the current logo and also use the background blue in the new logo instead of the existing blue. also probably makes sense to just use the logo instead of the old logo and ZeroBreak word in the app itself if you know what I mean

---

## Implementation Plan

### Summary

The new logo (`images/new-logo.jpg`) is a square "ZB" lettermark on a dark navy background — a much more polished brand mark than the current wavy-path `logo.png`. Three things need to change:

1. **Swap the logo image** — replace `logo.png` with `new-logo.jpg` everywhere it’s referenced.
2. **Remove the "ZeroBreak" heading text** from the header — the logo itself carries the brand, no need to repeat the name in text next to it.
3. **Update the app’s blue palette** — the new logo’s background is a slightly brighter/richer navy (approximately `#1a3057`) compared to the existing `--green-dark` (`#0e1b3d`) / `--green-mid` (`#192d68`). The header gradient should be updated to match.

---

### Files to Change

#### 1. `src/pages/LookupPage.tsx` — Header brand block

- Change `src="images/logo.png"` → `src="images/new-logo.jpg"`
- Remove the `<h1>ZeroBreak</h1>` element that sits next to the logo
- The logo image can be made slightly larger (e.g. `2.5rem` instead of `2rem`) since it’s now the sole brand element

#### 2. `src/styles/global.css` — Color variables & logo style

- Update the two CSS custom properties that drive the header gradient:
- `--green-dark` from `#0e1b3d` → new value (see open question below)
- `--green-mid` from `#192d68` → new value (see open question below)
- Update `.header-logo` — the new logo is a square with its own rounded background built in, so `border-radius: 6px` may look doubled; consider removing or reducing it.

#### 3. `src/theme.ts` — Mantine theme palette

- Update the `navy` color array entries at indices `7` and `8` to match the updated `--green-mid` / `--green-dark` values so Mantine components stay in sync.

#### 4. `index.html` — Favicon & apple-touch-icon

- Change `href="images/logo.png"` → `href="images/new-logo.jpg"` for both the `<link rel="icon">` and `<link rel="apple-touch-icon">` tags.
- **Note:** browsers prefer `.png`/`.ico` for favicons; a `.jpg` will technically work in most browsers but is non-standard. Worth converting `new-logo.jpg` to a `.png` (or generating a proper `favicon.ico`) as a follow-up, or doing it as part of this task.

---

### Step-by-Step Implementation

1. Update `index.html` favicon links to point to `new-logo.jpg`.
2. In `LookupPage.tsx`, update the `<img src>` and remove `<h1>ZeroBreak</h1>`.
3. In `global.css`, update `--green-dark` and `--green-mid` to the new navy values, and tweak `.header-logo` sizing/border-radius.
4. In `theme.ts`, update the matching `navy` palette entries.
5. Smoke-test: run the dev server, check the header on mobile and desktop, verify the logo doesn’t disappear into the background (the logo has its own dark background — a small `box-shadow` or `outline` on `.header-logo` may help it pop against the dark header gradient).

---

### Open Questions for Human Review

1. **Exact blue value** — The new logo’s background blue looks like approximately `#1a3057` but I can’t pixel-pick it precisely from the image. Please confirm the exact hex (or share it), so the CSS variables are accurate rather than approximated.
2. **Logo size in the header** — Currently `2rem × 2rem`. Since it’s now solo branding (no text next to it), should it be bigger (e.g. `2.5rem` or `3rem`)? Or keep it small?
3. **Favicon format** — Should `new-logo.jpg` be converted to a `.png` for proper favicon use, or is `.jpg` fine for now?
4. **Other pages** — `CalibratePage` and `LogPage` have their own `log-header` / page headers but do not currently display the logo. Should the new logo be added there too for consistency, or leave them as-is?

---

## Implementation Summary

All changes have been implemented:

### `index.html`

- Updated `<link rel="icon">` and `<link rel="apple-touch-icon">` to point to `images/new-logo.png` (PNG converted from JPG)
- Set `<meta name="theme-color" content="#1a3057">` to match the new navy palette

### `src/pages/LookupPage.tsx`

- Swapped `images/logo.png` → `images/new-logo.png`
- Removed the `<h1>ZeroBreak</h1>` text element (logo now stands alone as the brand mark)

### `src/pages/CalibratePage.tsx`

- Added `<div className="header-brand"><img … /></div>` with the new logo to the `log-header` center column

### `src/pages/LogPage.tsx`

- Same logo treatment as CalibratePage — added to the `log-header` center column

### `src/styles/global.css`

- `--green-dark`: `#0e1b3d` → `#1a3057`
- `--green-mid`: `#192d68` → `#25487e`
- `.header-logo`: increased to `3rem × 3rem`, reduced `border-radius` from `6px` to `4px` (logo has its own built-in rounded corners)

### `src/theme.ts`

- `navy[7]`: `#25487e` (was `#192d68`) — mirrors `--green-mid`
- `navy[8]`: `#1a3057` (was `#0e1b3d`) — mirrors `--green-dark`

### `images/new-logo.png`

- Converted from `new-logo.jpg` to `new-logo.png` for proper favicon browser support