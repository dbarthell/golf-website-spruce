---
id: GOL-jcv8xr
type: feature
title: Game mode
status: In Review
---
# Game mode

need a way to play 18 holes so i can log all my putts for a single round. storing these rounds indefinitely would be part of the paid tier along with the green maps

---

## Implementation Summary

Game mode is fully implemented across 7 files. Here's what was built:

### New Files
- **[`src/hooks/useGameMode.ts`](code://src/hooks/useGameMode.ts)** — Wraps `usePuttLog` and adds persistent hole tracking (`puttlog-game-hole` localStorage key). Exports `startGame`, `endGame`, `advanceHole`, `goToHole`, `logGamePutt`, `currentHole`, and pass-throughs for the active round state.
- **[`src/pages/GamePage.tsx`](code://src/pages/GamePage.tsx)** — Three-state page: no active game (start CTA + completed round history), active game (18-hole scorecard with current hole highlighted), and end-round confirmation sheet. Tapping any logged hole opens a `HoleStatCard` drawer with per-hole miss breakdown.

### Modified Files
- **[`src/hooks/usePuttLog.ts`](code://src/hooks/usePuttLog.ts)** — Added `hole?: number` to `PuttEntry`; added `mode`, `startedAt`, `completedAt` to `PuttRound`; added `startGame`/`endGame`/`activeGameRoundId`/`activeGameRound`/`allGameRoundIds` exports; `addPutt` now accepts optional `roundId` arg (defaults to today's practice round).
- **[`src/App.tsx`](code://src/App.tsx)** — Added `<Route path="/game" element={<GamePage />} />`.
- **[`src/components/BottomNav.tsx`](code://src/components/BottomNav.tsx)** — `/game` routes now highlight the putting tab.
- **[`src/pages/LookupPage.tsx`](code://src/pages/LookupPage.tsx)** — Imports `useGameMode`. Header shows ⛳ **Start Round** button when no game active; shows **⛳ Hole N** chip (links to `/game`) when a game is in progress. Passes `gameContext` down to `LookupResultPanel`.
- **[`src/components/LookupResult.tsx`](code://src/components/LookupResult.tsx)** — Added `gameContext` prop. When active, "Log this putt" shows the current hole number. Post-log confirmation becomes **"Log another"** / **"Next hole →"** buttons instead of the 4-second fade. Practice mode behavior is fully unchanged.
- **[`src/pages/LogPage.tsx`](code://src/pages/LogPage.tsx)** — Added **"⛳ Game Rounds"** section at top with per-round cards that expand to show a per-hole scorecard grid + aggregate miss tendency. 2 completed rounds free; older ones show the existing premium upsell card.
- **[`src/styles/global.css`](code://src/styles/global.css)** — ~300 lines of new styles: game strip, scorecard rows (current/dimmed states), hole detail overlay, start section, end round sheet, post-log buttons, completed round cards, and the active-game header chip.

### Key Decisions Implemented
- Hole advancement is **manual** — "Log another" stays on the hole, "Next hole →" calls `advanceHole()`
- **No auto-advance** — the hole counter only changes when the user taps
- **End Round** is valid at any point (9-hole rounds are first-class)
- Active round is always accessible; 2 completed game rounds free, older ones locked
- Data is backward-compatible — existing practice rounds get `mode: 'practice'` on read via fallback

---

## Implementation Plan

> **Decisions locked in** (from review):
> - Hole advancement is **manual** — users log N putts per hole at their own pace, then tap "Next hole" themselves. Logging a 2-footer is optional; the system doesn't force it.
> - **"Start Round"** lives on the main lookup screen (LookupPage), not buried in the Log tab.
> - Per-hole stats include **mades/misses + miss-tendency breakdowns** (not just a putt count).
> - **Free tier**: active round always accessible; 2 completed game rounds free, older ones locked (paid).
> - **"End Round"** is valid at any point — 9-hole rounds are first-class. No separate "abandon" flow.

### Summary

Game mode is a structured putting-round tracker layered on top of the existing `usePuttLog` system. When active, it associates each logged putt with a hole number (1–18) and presents a per-hole scorecard. The round is explicitly started and ended by the user. Historical game rounds beyond a free-tier limit are a paid feature (matching the existing `FREE_ROUND_LIMIT` pattern for practice rounds).

The current logging system is "practice mode" — it auto-creates rounds by date and has no hole context. Game mode adds a parallel concept without breaking the existing flow.

---

### Data Model Changes — `src/hooks/usePuttLog.ts`

1. **Extend `PuttEntry`** — add an optional `hole` field:
   ```ts
   export interface PuttEntry {
     // ...existing fields...
     hole?: number; // 1–18, present only for game-mode putts
   }
   ```

2. **Extend `PuttRound`** — add a `mode` discriminator and game metadata:
   ```ts
   export interface PuttRound {
     roundId: string;
     mode: 'practice' | 'game';
     startedAt?: number;   // epoch ms, for game rounds
     completedAt?: number; // epoch ms, set when user ends the round
     entries: PuttEntry[];
   }
   ```

3. **Round ID scheme** — game rounds use a timestamp-based ID so multiple rounds can exist on the same day:
   - Practice rounds (existing): `YYYY-MM-DD`
   - Game rounds (new): `game-{timestamp36}` e.g. `game-lx9zk4f`

4. **Active game round** — persist `localStorage.setItem('puttlog-active-game', roundId)` so the active game survives app restarts.

5. **New hook exports:**
   ```ts
   startGame(): string          // creates a new game round, persists as active, returns its ID
   endGame(): void              // sets completedAt, clears active-game key
   activeGameRoundId: string | null
   activeGameRound: PuttRound | null
   ```

---

### New Page — `src/pages/GamePage.tsx`

Route: `/game`

**States:**

**A) No active game**
- "Start Round" button (primary CTA)
- History of completed game rounds — 2 free, rest behind premium upsell
- Each completed round shows: date, holes played, total putts, make %

**B) Active game — scorecard view**
- Header: `⛳ Hole 7 of 18` · total putts so far · **"End Round"** button (valid at any time, works for 9-hole rounds)
- 18-row scorecard (holes not yet reached are dimmed)
- Each row: hole number · putts logged · make % · miss-direction badge (e.g. "2 left") · tap to jump to that hole
- Current hole row is highlighted; tapping a past hole lets you review or add a forgotten putt

**C) After tapping "End Round"**
- Confirmation sheet: "End after N holes?" with a summary
- On confirm: sets `completedAt`, clears active game, navigates to the completed round's stat view

**Hole-level stat card** (shown when tapping a hole in the scorecard or in the completed round view):
- Putt list for that hole (distance, outcome, clock)
- Miss tendency bar (short/long/left/right %)
- Identical to the existing `RoundStats` pattern but scoped to one hole's entries

---

### New Hook — `src/hooks/useGameMode.ts`

Wraps `usePuttLog` and adds hole-tracking state:
```ts
export function useGameMode() {
  const { activeGameRoundId, activeGameRound, startGame, endGame, addPutt } = usePuttLog();
  const [currentHole, setCurrentHole] = useState(() => readCurrentHole()); // persisted

  function advanceHole() {
    const next = Math.min(18, currentHole + 1);
    setCurrentHole(next);
    localStorage.setItem('puttlog-game-hole', String(next));
  }

  function goToHole(n: number) {
    localStorage.setItem('puttlog-game-hole', String(n));
    setCurrentHole(n);
  }

  // Called by LookupResult's "Log this putt" when a game is active
  function logGamePutt(entry: Omit<PuttEntry, 'id' | 'timestamp' | 'hole'>) {
    if (!activeGameRoundId) return;
    addPutt({ ...entry, hole: currentHole }, activeGameRoundId);
  }

  return { activeGameRoundId, activeGameRound, currentHole, goToHole,
           advanceHole, logGamePutt, startGame, endGame };
}
```

**Hole persistence:** `currentHole` is stored in `localStorage('puttlog-game-hole')` so backgrounding the app mid-round doesn't lose your place. Cleared when the round ends.

**No auto-advance:** The hole never changes automatically. After logging a putt the user sees two options inline: **"Log another"** (stay on this hole) and **"Next hole →"** (calls `advanceHole`). This handles multi-putt holes naturally and lets users skip logging a tap-in if they want.

---

### Changes to Existing Files

#### `src/hooks/usePuttLog.ts`
- Add `hole?: number` to `PuttEntry`
- Add `mode`, `startedAt`, `completedAt` to `PuttRound`
- Add `startGame`, `endGame`, `activeGameRoundId`, `activeGameRound` exports
- `addPutt` accepts an optional second arg `roundId?: string` — if omitted it uses today's practice round (existing behavior); if provided it writes to that specific round

#### `src/pages/LookupPage.tsx`
- Import `useGameMode`
- **When no game is active:** show a subtle **"⛳ Start Round"** button in the header links row (next to the existing Log / Calibrate / Guide links). Tapping it calls `startGame()` and navigates to `/game`.
- **When a game is active:** replace that button with a tappable **"⛳ Hole N"** chip that links to `/game`. Same visual slot, no layout shift.
- Pass `currentHole`, `activeGameRoundId`, and `logGamePutt` down to `LookupResultPanel`

#### `src/components/LookupResult.tsx`
- New optional props: `gameContext?: { roundId: string; hole: number; onLogGamePutt: (entry) => void; onNextHole: () => void }`
- When `gameContext` is present, "Log this putt" routes to `onLogGamePutt` instead of the practice `onLogPutt`
- **Post-log confirmation** (replaces current 4-second fade): two buttons — **"Log another"** (stay on hole, reset picker) and **"Next hole →"** (calls `onNextHole` + navigate back if on `/game`)
- Without `gameContext`, behavior is identical to today (no regression for practice mode)

#### `src/components/BottomNav.tsx`
- No new tab — game mode lives in the existing putting tab's ecosystem
- Add `'/game'` to the putting tab's `subPaths` so the bottom nav stays highlighted on `/game`

#### `src/App.tsx`
- Add `<Route path="/game" element={<GamePage />} />`

#### `src/pages/LogPage.tsx`
- Add a **"Game Rounds"** section at the top, above the existing practice-round selector
- Each completed game round shows: date, holes played, total putts, overall make %
- Tapping a game round expands a per-hole scorecard + aggregate stats (same miss-tendency and distance-bucket cards as practice, but also a hole-by-hole grid)
- 2 completed game rounds free; a `log-upsell-card` (matching existing style) gates the rest

---

### Free vs. Paid Tier

Per the artifact: _"storing these rounds indefinitely would be part of the paid tier."_

- **Active round**: always fully accessible regardless of tier
- **Completed game rounds**: **2 free**, older ones locked — matches the existing `FREE_ROUND_LIMIT` pattern and its existing upsell card UI
- The upsell copy in `LogPage` already mentions "CSV/JSON export with Premium" — this will extend to "Unlimited round history + CSV export"

---

### Implementation Sequence

1. **Extend data model** in `usePuttLog.ts` — add fields, new round-ID scheme, `startGame`/`endGame`, optional `roundId` param on `addPutt`
2. **`useGameMode.ts`** — new hook wrapping game-specific state
3. **`GamePage.tsx`** — new page with start/end UI and scorecard
4. **Wire `GamePage` into App router and BottomNav**
5. **Update `LookupPage` + `LookupResult`** — show active-game context, route log-putt to game round when active
6. **Update `LogPage`** — render game rounds with per-hole scorecard cards

---

## Decisions Log

| # | Question | Decision |
|---|----------|----------|
| 1 | Hole advancement | **Manual only.** Post-log shows "Log another" / "Next hole →". No auto-advance. |
| 2 | "Start Round" location | **Main lookup screen** — in the header links row. |
| 3 | Per-hole stats depth | **Full breakdown** — mades/misses + miss-tendency per hole, not just putt count. |
| 4 | Free tier boundary | **Active round always free; 2 completed rounds free**, rest locked. |
| 5 | Partial rounds | **"End Round" is valid at any point.** 9-hole rounds are first-class, no abandon flow. |