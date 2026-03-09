---
id: GOL-kasvw9
type: task
title: UX bugs
status: Done
---
# UX bugs

Now the “ inches label has disappeared after the big main backswing number. And it also seems to have gone back to decimals instead of rounding to nearest .5 to go 1 foot past the hole like it did before. Also when swithcing to cm, the big main lateral aim gets cut off — you can’t see the beginning of the number or the “out” label after cm. it also is in decimals. everything should be rounded to nearest 0.5 like we had before

## Implementation

Fixed four UX bugs in two files:

### `src/hooks/useUnits.ts`
- Added a shared `r05()` helper that rounds to the nearest 0.5 (using `Math.round(n * 2) / 2`).
- `fmtAim` in cm mode: switched from `.toFixed(1)` to `r05()` — now rounds to nearest 0.5 cm.
- `fmtBackswing` in cm mode: same fix — now rounds to nearest 0.5 cm.
- `fmtAimVariance` in cm mode: same fix — consistent rounding.

### `src/components/LookupResult.tsx`
- **Clock mode backswing**: Was rendering raw `{backswing}` (a bare number) — no `”` label and no rounding. Fixed to `{backswing !== null ? fmtBackswing(backswing) : ‘—‘}`, matching the straight-mode path.
- **Edge aim layout in cm**: `fmtAim(edgeAim)` was returning `”8.5 cm”` as a single text node at 2.5 rem, then `”out”` at 1 rem — causing the combined text to overflow its container (beginning of number clipped). Fixed by splitting `”cm”` into its own `<span className=”result-unit”>` so only the numeric value renders at large size, consistent with how `”out”` is already handled.