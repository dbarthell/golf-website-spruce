---
id: GOL-z8zxhc
type: feature
title: Streak counter and Made flash
status: Backlog
---
# Streak counter and Made flash

## What was built

### Streak counter
Tracks consecutive "Made" putts from today's round. Only counts putts ≥ 5 ft (tap-ins are ignored). After logging a make that extends a streak to 2+, a 🔥 badge animates in below the confirmation: "🔥 N in a row!".

### Made flash
When "Made" is tapped, the result card briefly pulses a gold border (`--green-accent`) and fades back over 600ms.

## Files changed
- [src/hooks/usePuttLog.ts](code://src/hooks/usePuttLog.ts) — added `computeStreak`, `currentStreak`, `STREAK_MIN_DISTANCE`
- [src/components/LookupResult.tsx](code://src/components/LookupResult.tsx) — streak badge, Made flash
- [src/pages/LookupPage.tsx](code://src/pages/LookupPage.tsx) — passes `currentStreak` to `LookupResultPanel`
- [src/styles/global.css](code://src/styles/global.css) — `streak-pop` and `made-flash` keyframes and classes