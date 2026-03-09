---
id: GOL-zblv4r
type: feature
title: ZBL Variance Display
priority: Medium
size: S
status: In Review
tasks: []
---
# ZBL Variance Display

Surface the `plusInches` / `minusInches` variance values already returned by `calculateZBLVector` in the straight-putt result panel. Today these values are computed but silently discarded.

## Background

`calculateZBLVector` already returns `{ aimInches, plusInches, minusInches }`. In straight mode the aim is shown but the variance band is never threaded through to the UI. This is a plumbing task — no new math required.

## Acceptance Criteria

- In `straight` mode, the result panel shows the aim as `3" ±1"` (or `3" +1" / −0.5"` if asymmetric) directly beneath the ZBL aim value
- Variance is only shown when `plusInches` / `minusInches` are non-null
- `clock` mode is unaffected (no variance display needed there per the decision in the memo)
- The display format is inline text — no SVG changes required

## Implementation Summary

Three files were changed to surface the `plusInches` / `minusInches` variance values in the straight-putt result panel:

1. [types.ts](code://src/lib/types.ts) — Added `zblPlus: string | null` and `zblMinus: string | null` optional fields to the `straight` variant of `LookupResult`.
2. [LookupPage.tsx](code://src/pages/LookupPage.tsx) — In `computeResult`, threaded `zblData.plusInches` and `zblData.minusInches` through as `zblPlus` / `zblMinus` in the returned straight result object.
3. [LookupResult.tsx](code://src/components/LookupResult.tsx) — In the straight mode branch, computed a `varianceLine` string: `±N"` when symmetric, `+N" / −M"` when asymmetric. Rendered it below the ZBL aim label using a new `.result-variance` CSS class (added to `global.css`) when non-null. Clock mode is untouched.

## Key Files

- Edit: [types.ts](code://src/lib/types.ts) — add optional `zblPlus` / `zblMinus` fields to the `straight` variant of `LookupResult`
- Edit: [LookupPage.tsx](code://src/pages/LookupPage.tsx) — thread variance fields through `computeResult`
- Edit: [LookupResult.tsx](code://src/components/LookupResult.tsx) — render `±` line under aim value