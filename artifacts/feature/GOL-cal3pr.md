---
id: GOL-cal3pr
type: feature
title: Multiple Calibration Profiles
status: Backlog
priority: Medium
size: M
tasks: []
---
# Multiple Calibration Profiles

Support named calibration profiles (e.g., "Home Course", "Range Practice", "Fast Bermuda") so golfers can switch contexts without re-calibrating every time.

## Background

Green speed and personal stroke feel vary meaningfully between home course, away events, and practice greens. Right now there is a single `distanceFactor` in `localStorage`. Named profiles let users pre-calibrate for their common contexts and switch instantly.

## Acceptance Criteria

- `useCalibration` stores `{ profiles: Record<string, Calibration>, active: string }` instead of a bare `Calibration`
- On first load, if the old `putt-cal` key exists, migrate it to a `"default"` profile automatically (no data loss)
- `CalibratePage` shows a profile selector row at the top (chips or `<select>`)
- "New Profile" button opens a name input and creates a new profile (copies current values as a starting point)
- Switching profiles immediately updates the active calibration used by `LookupPage`
- Profile names can be renamed or deleted (with a guard: cannot delete the last profile)
- A `CalibrationProfile` type is added to `types.ts`

## Key Files

- Edit: [useCalibration.ts](code://src/hooks/useCalibration.ts)
- Edit: [types.ts](code://src/lib/types.ts) — add `CalibrationProfile`
- Edit: [CalibratePage.tsx](code://src/pages/CalibratePage.tsx) — profile selector UI
