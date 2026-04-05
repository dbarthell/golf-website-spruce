---
id: GOL-clkpol1
type: feature
title: Putting Clock & Wedge Gauge Polish
status: Done
---
# Putting Clock & Wedge Gauge Polish

A set of visual refinements across the putting clock face and wedge gauge.

## Implemented Changes

### Putting Clock Face (`ClockFace.tsx`)

1. **Tilt more subtle** — reduced 3-D perspective tilt from `slope × 5°` to `slope × 2°` (max 12° at 6% slope instead of 30°).
2. **Gold target ring removed** — removed the gold aim ring and inner dot from the SVG overlay. Also removed the matching inline gold ring SVG icon from the "Aim" result label in `LookupResult.tsx`.
3. **ZBL label split** — "ZBL" stays centered above the dot (`y = zblY - 12`, `textAnchor="middle"`); the inches value sits beside the vertical measurement line at its midpoint (`y = (SVG_C + zblY) / 2`), flipping to the left side when a right-side clock position is selected (`sin(θ) > 0`).

### Wedge Gauge (`WedgeClockFace.tsx`)

4. **Progress arc from ball** — gold bezel arc now starts at 180° (golf ball / 6 o'clock) instead of 225° (7:30), sweeping clockwise up to the selected position.
5. **Gap ticks** — added `isGap` flag for ticks at 195° and 210° (between golf ball and 7:30); rendered at opacity 0.28 instead of 0.07 so they visibly bridge the gap.

### Wedge Distance Chart (`WedgeMatrix.tsx`)

6. **Clock icon column header** — added `IconClock` (size 14) as the header for the position column.

### Wedge Distance Chart — Anchor Indicator

7. **Gold underline on anchor cells** — cells whose value is a measured anchor show a gold `border-bottom` underline via `.wedge-cell-inner--anchor`. The `.wedge-cell-inner` wrapper ensures alignment is unaffected for non-anchor cells.

### Putting Module Headers (reverted)

- Experimented with larger font sizes (up to 2.5rem) and a white header style for Putt Log / Calibrate pages — reverted to original gradient header, 1.25rem h1, and original padding/height after review.