---
id: GOL-ee9tq5
type: task
title: MVCC logo
---
# MVCC logo

Let's replace the current golf emoji with the Minnesota Valley Country Club logo (the club located in Bloomington, MN) which is my home course

## Implementation Plan

### Current State

- The hero header in `index.html` (line 14) displays a golf flag emoji (`&#9971;` / ⛳) inside a `<div class="hero-icon">` element
- The `.hero-icon` class in `css/styles.css` sets `font-size: 3rem` — styled for emoji text, not an image
- The `images/` directory exists but is empty — no assets currently in the project

### MVCC Logo Source

- MVCC's website has a color logo at: `https://www.mvccgolf.com/wp-content/uploads/2022/04/MVCC-Logo-f_Clr-250.jpg`
- They also have a grayscale version at: `https://www.mvccgolf.com/wp-content/uploads/2021/11/MVCC-Logo-f_Clr-gray.jpg`

### Steps

1. **Download the MVCC logo** into `images/`

- Save the color logo locally as `images/mvcc-logo.jpg`
- This avoids hotlinking to their server and ensures the asset is always available

2. **Update&#32;`index.html`** — Replace the emoji div with an `<img>` tag

- Change `<div class="hero-icon">&#9971;</div>` → `<img src="images/mvcc-logo.jpg" alt="Minnesota Valley Country Club" class="hero-icon">`

3. **Update&#32;`css/styles.css`** — Restyle `.hero-icon` for an image instead of emoji text

- Remove `font-size: 3rem` (not applicable to images)
- Add `width: 100px; height: auto;` to size the logo appropriately in the hero
- Add `border-radius: var(--radius-md);` for polish
- Keep `margin-bottom: var(--space-sm)` as-is

### Notes

- The logo is a JPG with a background — if we want a cleaner look we could explore whether a transparent PNG/SVG version is available, but the JPG should work well on the dark green gradient hero
- No other files reference the golf emoji, so this is a single-location change