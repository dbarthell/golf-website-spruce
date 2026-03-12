---
id: GOL-g4ps1g
type: feature
title: More UI enhancements
status: Done
---
# More UI enhancements

1. green stimp selector on main app doesn’t need to be so tall. that will give us a little more room if we need it in the future
2. green speed selector on the calibrate screen is a little hard to see because it’s white on a gray/beige background
3. let’s move the uphill/downhill line below Backswing on the left
4. let’s move the L -> R after AIM on the same line under the big lateral aim number instead of on it’s own line after uphill/downhill

## Implementation

**Fix 1 — Stimp selector height (main app):** Added `size="sm"` to the Mantine `SegmentedControl` when the `variant` is `’dark’` in `StimpToggle.tsx`. This gives the hero panel stimp toggle a more compact height while leaving the calibrate page toggle at default `’md’` size.

**Fix 2 — Calibrate page stimp selector contrast:** Updated `.stimp-sc-indicator-light` in `global.css` to use `var(--green-dark)` (navy) as the active indicator background instead of white, and changed `.stimp-sc-label-light[data-active]` to white text. Non-active label color bumped from `var(--gray-600)` to `var(--gray-800)` for better legibility on the light card background.

**Fix 3 — Uphill/downhill below Backswing (straight mode):** Removed the separate `.slope-row` at the bottom of the straight-mode result panel. Instead, the ↑/↓ distance and backswing values are now rendered inline inside the left (Backswing) `result-item` using a new `.slope-inline` CSS class that stacks them vertically below the "Backswing" label.

**Fix 4 — L→R direction inside Aim column (clock mode):** Removed the standalone L→R `.slope-row` at the bottom of the clock-mode result panel. The break direction (`L→R`, `R→L`) is now rendered directly inside `aimCell` below the ZBL reference line using a new `.result-break-dir` class. Uphill/Downhill distance is similarly moved into the Backswing column using `.slope-inline-centered`.

The `.result-divider` was also updated to use `align-self: stretch` (with `min-height: 60px`) so it correctly spans the height of both columns when one grows taller than the other.