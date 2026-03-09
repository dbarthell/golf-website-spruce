---
id: GOL-log8rp
type: feature
title: Round Putt Log
priority: High
size: M
status: Done
tasks: []
---
# Round Putt Log

A lightweight session log that lets golfers record each putt during a round and review miss tendencies at the end. Turns ZeroBreak from a lookup tool into a practice and analytics companion.

## Background

Most golfers have no idea how they actually putt. A simple log anchored to the app's existing distance/slope/stimp inputs creates an effortless record-keeping flow on the course.

## Data Shape

```ts
interface PuttEntry {
  id: string;          // uuid-lite
  timestamp: number;
  distance: number;
  slope: number;
  stimp: number;
  clock: string | null;
  outcome: 'made' | 'short' | 'long' | 'left' | 'right';
}
```

Entries are stored as a JSON array in `localStorage` with a `roundId` (date-based, e.g. `"2026-03-08"`) so data doesn't bleed across sessions.

## Acceptance Criteria

- A new `usePuttLog` hook persists entries to `localStorage`, grouped by `roundId`
- The result panel shows a "Log this putt" button that opens an inline outcome picker (made / miss direction)
- Tapping an outcome saves the current distance/slope/stimp/clock + outcome as a new entry
- A new `/log` route renders `LogPage`
- `LookupPage` header links to `/log` (alongside the Calibrate icon)
- `LogPage` shows:
- Putts by distance bucket (e.g., 0–10 ft, 10–20 ft, 20–30 ft, 30+ ft)
- Miss tendency breakdown (% made, % short, % long, % left, % right)
- Entry count and date for the current round
- **Free tier:** Shows the last 3 rounds only
- **Premium hook:** Full history + CSV/JSON export unlocked at a premium tier (stub the gate — no payment integration required yet, just a "Unlock Full History" upsell card)

## Implementation Notes

### What was built

**`src/hooks/usePuttLog.ts`** — New hook managing localStorage persistence for putt entries. Each round is stored under a key like `puttlog-round-2026-03-08`, with a `puttlog-index` key tracking which round IDs exist. Exports `PuttEntry`, `PuttOutcome`, `PuttRound` types plus `todayRoundId()` helper. The hook provides `addPutt`, `allRoundIds`, `currentRound`, and `getRound()`.

**`src/pages/LogPage.tsx`** — New `/log` route page. Shows:

- A round selector chip row (Today + up to 2 previous rounds = 3 free-tier rounds)
- Round summary card (total putts, make %, made count)
- Distance bucket breakdown (0–10, 10–20, 20–30, 30+ ft) with make-rate progress bars
- Miss tendency grid showing % and count for each outcome
- A premium upsell card ("Unlock Full History") when the user has > 3 rounds logged, with a stubbed Upgrade button

**`src/components/LookupResult.tsx`** — Added inline outcome picker:

- "Log this putt" pill button appears at the bottom of the result panel whenever a valid distance is set
- Tapping opens a 5-button outcome row (Made ✓ / Short ↓ / Long ↑ / Left ← / Right →)
- Tapping an outcome calls `onLogPutt()` with the full putt context, then shows a 2-second "Logged: {outcome}" confirmation
- A ✕ button dismisses the picker without logging

**`src/pages/LookupPage.tsx`** — Added Log link in the header (next to Calibrate), using `IconClipboardList`. Instantiates `usePuttLog` and passes `addPutt` + `puttContext` down to `LookupResultPanel`.

**`src/App.tsx`** — Added `/log` route pointing to `LogPage`.

**`src/styles/global.css`** — Added styles for: `.log-putt-section`, `.log-putt-btn`, `.outcome-picker`, `.outcome-btn`, `.outcome-saved`, `.log-round-selector`, `.log-round-chip`, `.log-card`, `.log-summary-*`, `.log-bucket-*`, `.log-outcome-*`, `.log-empty`, `.log-upsell-card`.

### Free vs. Premium

- Free tier: shows today + up to 2 prior rounds (3 total) in the round selector
- Premium gate: a stubbed "Unlock Full History" card appears when >3 rounds are present; no payment integration wired

## Key Files

- New: [usePuttLog.ts](code://src/hooks/usePuttLog.ts)
- New: [LogPage.tsx](code://src/pages/LogPage.tsx)
- Edit: [App.tsx](code://src/App.tsx) — add `/log` route
- Edit: [LookupPage.tsx](code://src/pages/LookupPage.tsx) — "Log this putt" button + log header link
- Edit: [LookupResult.tsx](code://src/components/LookupResult.tsx) — outcome picker UI