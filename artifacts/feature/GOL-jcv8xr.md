---
id: GOL-jcv8xr
type: feature
title: Live Round & Stats
status: In Review
---
# Live Round & Stats

## Summary

Built a full Live Round tracker and replaced the old practice log with an aggregate Stats dashboard.

---

## Live Round (`/game`)

New page at [GamePage.tsx](code://src/pages/GamePage.tsx) with hole-by-hole round tracking:

- Start and end rounds with confirmation sheets
- 18-hole scorecard with per-hole putt tallies
- **Quick count entry** — tap 1–5 to set putts for a hole and auto-advance; last putt marked made, rest short
- **Detailed putt entry** — log distance, slope, stimp, clock position, and outcome from the lookup tool; add from hole drawer or directly from the clock face screen
- Tapping "Made" on the clock face auto-advances to the next hole (with green flash)
- Tapping "Next hole" after a miss auto-logs a made putt for the current hole
- Expandable hole stat drawer with quick count, deletable detailed putts, and add detailed putt form
- Delete round (with confirmation)
- Edit holes after completing a round
- Completed rounds card with miss tendency breakdown
- Locked older rounds collapsed into a single upsell card

---

## Hooks

[usePuttLog.ts](code://src/hooks/usePuttLog.ts) — Extended with:

- `hole?: number` on `PuttEntry`
- `mode`, `startedAt`, `completedAt` on `PuttRound`
- `startGame()`, `endGame()`, `deleteGameRound()`
- `activeGameRoundId`, `activeGameRound`, `allGameRoundIds`
- `addPutt` accepts optional `targetRoundId`

[useGameMode.ts](code://src/hooks/useGameMode.ts) — New hook wrapping `usePuttLog` with `currentHole` state persisted to localStorage, plus `advanceHole`, `goToHole`, `logGamePutt`, and `startGame`.

---

## Stats Module (`/log`)

[LogPage.tsx](code://src/pages/LogPage.tsx) replaced the old practice log with an aggregate stats dashboard over completed rounds:

| Card | What it shows |
| --- | --- |
| Summary | Putts/round · 1-putt % · 3-putt % |
| Putt Distribution | 1 / 2 / 3+ putt breakdown with bars |
| Make % by Distance | <5′ · 5–10′ · 10–15′ · 15–20′ · 20–25′ · >25′ |
| Miss Tendency | Short / long / left / right % (detailed putts only) |
| By Hole | Horizontally scrollable bar chart, color-coded ≤1.5 green / ≥2.5 red |

Quick-entry putts (distance = 0) are excluded from make % and miss tendency to avoid skewing results.

---

## Other changes

- [LookupPage.tsx](code://src/pages/LookupPage.tsx) — Removed practice logging; header link updated to "Stats" with bar chart icon; flag icon navigates to `/game` without auto-starting a round; active game chip shows current hole
- [LookupResult.tsx](code://src/components/LookupResult.tsx) — Removed `onLogPutt` and `streak` props; practice-mode post-log UI removed; game-mode logging retained with auto-advance on made
- [global.css](code://src/styles/global.css) — \~400+ lines of new styles for game mode, scorecard, quick count, stats page charts, and hole legend