---
id: GOL-hpt7mx
type: feature
title: Haptic Feedback on Mobile
status: Backlog
priority: Low
size: S
tasks: []
---
# Haptic Feedback on Mobile

Add short haptic pulses via `navigator.vibrate` at key interaction moments so golfers on the course get tactile confirmation without looking up from the screen.

## Acceptance Criteria

- A new `src/lib/haptics.ts` module wraps `navigator.vibrate` with a feature-detect guard (no-op on unsupported devices)
- A short pulse fires when the lookup result updates (distance/slope/stimp change)
- A pulse fires when the user taps a clock position on the `ClockFace`
- A pulse fires when calibration is saved on `CalibratePage`
- No errors or warnings on desktop browsers that don't support `navigator.vibrate`

## Key Files

- New: [haptics.ts](code://src/lib/haptics.ts)
- Edit: [LookupPage.tsx](code://src/pages/LookupPage.tsx) — `useEffect` on result change
- Edit: [ClockFace.tsx](code://src/components/ClockFace.tsx) — onClick handler
- Edit: [CalibratePage.tsx](code://src/pages/CalibratePage.tsx) — save button handler
