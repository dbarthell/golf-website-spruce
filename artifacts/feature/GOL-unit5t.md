---
id: GOL-unit5t
type: feature
title: Distance Units Toggle (ft ↔ m)
priority: Medium
size: M
status: Done
tasks: []
---
# Distance Units Toggle (ft ↔ m)

Add a global setting to display distances in metres instead of feet. All internal calculations remain in feet; only display values are converted. In metric mode, ZBL aim output (inches) also converts to centimetres for consistency.

## Background

Golfers outside the US measure distances in metres. The conversion is simple (`1 ft = 0.3048 m`, `1 in = 2.54 cm`), but the unit value needs to be threaded through every component that shows a distance or aim value.

## Acceptance Criteria

- A new `useUnits` hook persists `'ft' | 'm'` to `localStorage`
- A global toggle (header or settings chip) switches between units instantly
- `DistanceInput` shows the correct label ("ft" vs "m") and converts on input/output
- `CommonDistances` preset labels convert to metres in metric mode
- `BackswingTable` distance column converts to metres in metric mode
- `CalibratePage` test-distance chips and preview table convert to metres
- ZBL aim output converts from inches to centimetres in metric mode
- Underlying calculations are never changed — conversion happens only at display boundaries

## Conversion Reference

- Distance: `metres = feet × 0.3048`; `feet = metres / 0.3048`
- Aim: `cm = inches × 2.54`

## Implementation Summary

### New files

- **`src/context/UnitsContext.tsx`** — React context holding `unit: 'ft' | 'm'` and `toggleUnit`. Persists the preference to `localStorage` under the key `zerobreak-units`. Wraps the entire app via `UnitsProvider`.
- **`src/hooks/useUnits.ts`** — Consumes `UnitsContext` and exposes helper functions: `fmtDist(feet)`, `fmtAim(inches)`, `fmtAimVariance(inchesStr)`, `ftToDisplay(feet)`, `displayToFt(val)`, and the `unitLabel` / `aimLabel` strings. Also exports standalone conversion utilities (`ftToM`, `mToFt`, `inToCm`).

### Modified files

- **`src/App.tsx`** — Wrapped `<Routes>` in `<UnitsProvider>` so the unit state is shared across all pages.
- **`src/lib/types.ts`** — Added `zblRaw: number` to the `straight` variant of `LookupResult` so `LookupResultPanel` has the raw inches value available for metric conversion.
- **`src/pages/LookupPage.tsx`** — Imports `useUnits`, adds `zblRaw` to the straight result object, and renders a `unit-toggle` button in the header alongside the Calibrate link.
- **`src/components/DistanceInput.tsx`** — Now unit-aware: displays value in the current unit (metres or feet), converts on input and output so the parent always works in feet. Min/max/step adjust for metric mode.
- **`src/components/CommonDistances.tsx`** — Uses `fmtDist` to label each preset card in the current unit (e.g. "3.0 m" instead of "10 ft"). The `onSelect` callback still passes the underlying feet value.
- **`src/components/BackswingTable.tsx`** — Distance column converted via `fmtDist`; backswing column (inches) is unchanged.
- **`src/pages/CalibratePage.tsx`** — Imports `useUnits`, adds a `unit-toggle` button to the Calibrate header, and converts test-distance chip labels and preview-table distance column via `fmtDist`.
- **`src/components/LookupResult.tsx`** — ZBL aim display, variance lines, flat-putt slope note, uphillTotal/downhillTotal distances, and clock-mode adjDist all render in the current unit. Aim values use `fmtAim` (inches → cm in metric); distances use `fmtDist`. All internal `LookupResult` data remains in feet/inches.
- **`src/styles/global.css`** — Added `.unit-toggle` button styles (pill badge in the dark header, consistent with the Calibrate link style).

### Conversion constants used

- Distance: `1 ft = 0.3048 m`
- Aim: `1 in = 2.54 cm`

All calculations remain in imperial (feet/inches) throughout. Conversion happens only at display boundaries inside the components.

## Key Files

- New: [useUnits.ts](code://src/hooks/useUnits.ts)
- Edit: [DistanceInput.tsx](code://src/components/DistanceInput.tsx)
- Edit: [CommonDistances.tsx](code://src/components/CommonDistances.tsx)
- Edit: [BackswingTable.tsx](code://src/components/BackswingTable.tsx)
- Edit: [CalibratePage.tsx](code://src/pages/CalibratePage.tsx)
- Edit: [LookupResult.tsx](code://src/components/LookupResult.tsx) — aim output in cm
- Edit: [App.tsx](code://src/App.tsx) or header component — add unit toggle button