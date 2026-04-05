---
id: GOL-shzhw5
type: feature
title: Wedge Clock Module — Gauge, Calibration & Distance Chart
status: Done
---
# Wedge Clock Module — Gauge, Calibration & Distance Chart

I want to create a separate module for calibrating, calculating, and visuazling yardages with wedges based on the clock system. Many pros like Russell Henley calibrate their wedge yardages based on where their left arm is on a clock (e.g. 7:30, 9, 10, 10:30, etc.).

---

## Implementation Plan

### Decisions

| # | Question | Decision |
| --- | --- | --- |
| Units | Yards only or toggle? | **Yards only** |
| Clock arc | Full ring or partial arc? | **Arc only: 7:30, 8:00, 9:00, 10:00, 10:30, Full** |
| Handedness | Right or left support? | **Right-handed only** |
| Club defaults | Which clubs? | **LW, SW, GW** (no PW) |
| Nav | Header icon or separate? | **Separate nav entry point** |
| Interpolation | Snap or interpolate? | **Interpolate — suggest fractional position between two known ones** |
| Calibration UX | Manual grid or single-point anchor? | **Single-point anchor with auto-derived table; manual override per position** |
| Overview | Per-club view only? | **All-wedges matrix table as a dedicated section** |

---

### Summary

A brand-new, self-contained module alongside the existing putting app. The wedge clock system maps backswing arm positions to carry distances for each wedge. The user anchors one position per club (e.g. "GW at 9:00 = 60 yds"), the app derives all other positions using standard percentage ratios, and the user can fine-tune individual positions if needed.

The main lookup page shows an interactive clock arc for a selected club, plus an all-wedges matrix for a full-bag overview. The reference image confirms the visual direction: bold clock ring, arm hand pointing to position, yardage displayed prominently.

---

### Clock Positions (6 total)

| Key | Label | % of Full | Description |
| --- | --- | --- | --- |
| `7.5` | 7:30 | 55% | Short chip |
| `8` | 8:00 | 65% |  |
| `9` | 9:00 | 75% | Half swing — arm parallel |
| `10` | 10:00 | 85% |  |
| `10.5` | 10:30 | 92% | Three-quarter swing |
| `full` | Full | 100% | Full swing (12:00) |

These percentages are the **baseline ratios** used to extrapolate from a single anchor point. Example: anchor GW at 9:00 = 60 yds → full = 60 / 0.75 = 80 yds → 7:30 = 80 × 0.55 = 44 yds, etc. All derived values are rounded to the nearest whole yard and stored. The user can manually override any individual position after derivation.

---

### Data Model (`src/lib/wedge.ts` — new file)

```typescript
// Fixed club set (no custom clubs)
type WedgeClubId = 'lw' | 'sw' | 'gw';

interface WedgeClub {
  id: WedgeClubId;
  label: string;  // "LW", "SW", "GW"
}

// One data point: a club at a clock position
interface WedgeEntry {
  clubId: WedgeClubId;
  clockPosition: WedgePosition;  // '7.5' | '8' | '9' | '10' | '10.5' | 'full'
  yards: number;
  derived: boolean;  // true = computed from anchor, false = manually set
}

interface WedgeCalibrationData {
  entries: WedgeEntry[];
  // anchor per club: the one position the user actually measured
  anchors: Partial<Record<WedgeClubId, WedgePosition>>;
}
```

**Storage:** `localStorage` key `wedge-cal`.

**Key logic in&#32;`wedge.ts`:**

- `CLOCK_POSITIONS` — ordered array with keys, labels, and baseline ratios
- `deriveAllFromAnchor(clubId, anchorPosition, anchorYards)` → `WedgeEntry[]` — computes full set
- `findBestMatch(targetYards, entries)` → `{ clubId, clockPosition, yards, interpolated?: { from, to, label } }[]` — finds closest match(es), with interpolation between two adjacent positions when the target falls between them
- `interpolatePosition(pos1, pos2, t)` → display label like "between 9:00 and 10:00"

---

### New Files

| File | Purpose |
| --- | --- |
| `src/lib/wedge.ts` | Types, clock position constants + ratios, derivation logic, best-match + interpolation |
| `src/hooks/useWedgeCalibration.ts` | localStorage read/write, exposes derived + override entries |
| `src/components/WedgeClockFace.tsx` | Clock arc visual (arm hand + yardage badge, interactive or read-only) |
| `src/components/WedgeMatrix.tsx` | All-clubs × all-positions overview table |
| `src/pages/WedgePage.tsx` | Main lookup page (clock + matrix) |
| `src/pages/WedgeCalibratePage.tsx` | Single-point anchor entry + per-position override grid |

---

### Files to Modify

| File | Change |
| --- | --- |
| `src/App.tsx` | Add `/wedges` and `/wedges/calibrate` routes |
| `src/index.html` or nav component | Add wedge entry point in separate nav |
| `src/css/global.css` | Wedge styles (or new `wedge.css` imported in `main.tsx`) |

---

### WedgeClockFace Component

- Renders a **partial arc** from 7:30 up through 12:00 (Full) — not a full ring
- 6 tappable position nodes along the arc, each labelled
- A single **arm hand** rotates to the active position
- Below the clock: bold yardage badge (e.g. **75 YDS**), or "—" if uncalibrated
- When interpolation is active (target falls between two positions): both adjacent nodes highlighted with the fractional label shown (e.g. "\~9:30")
- Two modes: `interactive` (tap to select) used on the lookup page; `calibrate` (shows override inputs inline) used on the calibrate page

---

### WedgePage (Lookup) Flow

```
┌─────────────────────────────────────────┐
│  [← Putting]           Wedges  [Calibrate →] │
├─────────────────────────────────────────┤
│  Target yardage: [ 65 yds ]             │  ← optional input
│                                         │
│  [LW]  [SW]  [GW]   ← club tabs        │
│                                         │
│        ╭── clock arc ──╮               │
│       10:30           Full             │
│      10                                │
│       9       ◉ hand                  │
│        8                               │
│         7:30                           │
│                                         │
│            ▼  75 YDS  ▼               │
├─────────────────────────────────────────┤
│  All Wedges                             │
│  ┌──────┬────┬────┬────┐               │
│  │      │ LW │ SW │ GW │               │
│  ├──────┼────┼────┼────┤               │
│  │ 7:30 │ 45 │ 52 │ 58 │               │
│  │ 8:00 │ 53 │ 61 │ 68 │               │
│  │ 9:00 │ 61 │ 70 │ 78 │  ← highlighted│
│  │10:00 │ 69 │ 80 │ 89 │               │
│  │10:30 │ 75 │ 87 │ 97 │               │
│  │ Full │ 82 │ 95 │105 │               │
│  └──────┴────┴────┴────┘               │
└─────────────────────────────────────────┘
```

- Club tabs control the clock view above; the matrix always shows all three
- If a target yardage is entered, the closest cell(s) in the matrix are highlighted
- Interpolated suggestions shown as "\~9:30" in the clock arc

---

### WedgeCalibratePage Flow

1. **Anchor section** (primary): For each club — one position picker (the 6-position arc) + one yardage input. Tap "Derive all" and the full table is computed.
2. **Override section** (secondary, collapsed by default): Shows the derived table for each club with editable fields — user can tap any cell to manually override a specific position.
3. Save persists to localStorage; derived entries are flagged so they can be re-derived if the anchor changes.

---

### Interpolation Logic

When `findBestMatch(targetYards, entries)` is called and the target falls between two adjacent calibrated positions for the same club:

- Compute `t = (target - yardsLow) / (yardsHigh - yardsLow)`
- Return a result with `interpolated: { from: pos1, to: pos2, fraction: t, label: "~9:30" }` 
- The clock face hand renders at the interpolated angle
- The label reads e.g. **"\~9:30 swing"** with the target yardage shown

---

### Insights from Russell Henley Interview

These directly affect the design:

**1. Think in 5-yard increments, not exact yardages.**
Russell doesn't try to dial in to 97 yards — he picks the nearest 5-yard "feel" (90, 95, 100, 105) and commits. The app should reinforce this mental model. The matrix should show yardages but the lookup result framing should be "your 95-yard swing" rather than "93.4 yards."

**2. Clock positions map to \~5-yard steps.**
He explicitly describes: 9:00 = 90, \~9:30 = 95, 10:00 = 100, 10:30 = 105. This validates our interpolation model — a half-position between two nodes is a real, usable shot, not an edge case.

**3. "Flight" version = same clock position, ball back \~half a ball.**
For spin/trajectory control (into wind, firm greens, rough), Russell moves the ball back slightly and swings the same clock position. This produces \~5 yards less carry with a lower, less-spinning flight. This is a first-class variation to model, not just a footnote.

**Impact on data model:** Each clock position entry should optionally carry a `flightYards` value alongside `yards`. The lookup can then offer two rows per match: "GW 9:00 — 75 yds" and "GW 9:00 flight — 70 yds." The calibration UI should let the user enter both (or derive flight as `yards - 5` as a default).

**4. "Derive hint" is important.**
Derived entries should show a subtle indicator ("est.") so the user knows which numbers they've actually verified vs. which were calculated. This builds trust and reminds them to go verify.

**5. The "guess then check" training loop (Jim McLean drill).**
After hitting, guess your distance, then see the actual. This is a training behavior, not just logging. V1 of the wedge log could be: hit a shot → enter your guess → reveal actual (manual entry) → app tracks your calibration accuracy over time. This is a future feature but worth noting now so the data model leaves room for it.

**6. Situation-based club selection.**
Russell explicitly picks a lower-lofted club (51° instead of 55°) when he needs to take spin off. The matrix view naturally supports this — you can look across the row and see "which club gets me to 100 yards on a more controlled swing."

---

### Updated Data Model (with `flightYards`)

```typescript
interface WedgeEntry {
  clubId: WedgeClubId;
  clockPosition: WedgePosition;
  yards: number;         // normal carry — neutral ball position
  flightYards?: number;  // flight version — ball back ~½ ball, lower/less spin
  derived: boolean;
}
```

When `flightYards` is not set, default to `yards - 5` (derived). The calibration page shows both fields; `flightYards` is optional and collapsed by default.

**Matrix display:** The matrix shows `yards` values by default. A "Show flight" toggle swaps all cells to `flightYards`.

---

### Updated WedgePage Layout

```
┌─────────────────────────────────────────┐
│  Wedges                    [Calibrate →] │
├─────────────────────────────────────────┤
│  Target yardage: [ 95 yds ]             │
│                                         │
│  [LW]  [SW]  [GW]                       │
│                                         │
│        ╭── clock arc ──╮               │
│       10:30           Full             │
│      10                                │
│       9:30    ◉ (interpolated)         │  ← "~9:30 swing"
│        9                               │
│         7:30                           │
│                                         │
│       ~9:30 swing  •  95 YDS           │
│         [also: GW 9:00 flight — 95]    │  ← flight variant shown inline
├─────────────────────────────────────────┤
│  All Wedges          [Normal] [Flight]  │  ← toggle
│  ┌──────┬──────┬──────┬──────┐          │
│  │      │  LW  │  SW  │  GW  │          │
│  ├──────┼──────┼──────┼──────┤          │
│  │ 7:30 │  45  │  52  │  58  │          │
│  │ 8:00 │  53  │  61  │  68  │          │
│  │ 9:00 │  61  │  70  │  78  │          │
│  │~9:30 │  —   │  —   │  95* │ ← interp │
│  │10:00 │  69  │  80  │  89  │          │
│  │10:30 │  75  │  87  │  97  │          │
│  │ Full │  82  │  95  │ 105  │          │
│  └──────┴──────┴──────┴──────┘          │
└─────────────────────────────────────────┘
```

Note: interpolated rows in the matrix are shown only when a target yardage is active and falls between two positions. The `*` marker denotes an estimated value.

---

---

## Implementation Summary

### What Was Built

**New files:**

- **`src/lib/wedge.ts`** — Core types (`WedgeClubId`, `WedgePosition`, `WedgeEntry`, `WedgeCalibrationData`), `CLOCK_POSITIONS` constants with 6 positions (7:30→Full, with baseline ratios and arc angles), `deriveAllFromAnchor()` (computes all 6 yardages + flightYards from one anchor), and `findBestMatch()` with full interpolation logic (returns `InterpolationResult` with angle, fraction, and `~9:30`-style labels).
- **`src/hooks/useWedgeCalibration.ts`** — localStorage-backed hook (`wedge-cal` key) exposing `setAnchor`, `overrideEntry`, `rederive`, `resetClub`, `getClubEntries`, and `isCalibrated`. Single-anchor → full derivation pattern; manual overrides flip `derived: false`.
- **`src/components/WedgeClockFace.tsx`** — SVG arc visual from 7:30 to 12:00 with 6 tappable position nodes, rotating gold hand, active arc highlight, outer position labels, yardage badge below. Supports `interactive` and `display` modes, optional yardage values on nodes, and interpolated hand placement.
- **`src/components/WedgeMatrix.tsx`** — All-clubs × all-positions table. Normal/flight toggle. Highlights rows/cells matching target yardage. Inserts a virtual interpolated row (e.g. `~9:30`) between bracketing positions when a target is active. Derived cells marked with `est`.
- **`src/pages/WedgePage.tsx`** — Main lookup page: target yardage input → club tabs (LW/SW/GW) → clock arc with interpolated hand → yardage badge → flight variant hint → All Wedges matrix with Normal/Flight toggle.
- **`src/pages/WedgeCalibratePage.tsx`** — Per-club calibration cards: 6-position picker → yardage input → "Derive All" button. Collapsed "Fine-tune individual positions" override grid with Carry + Flight columns and anchor/est./manual source badges. Russell Henley 5-yard-increments tip at bottom.

**Modified files:**

- **`src/App.tsx`** — Added `/wedges` and `/wedges/calibrate` routes.
- **`src/pages/LookupPage.tsx`** — Added Wedges nav link (bolt icon) in the header.
- **`src/styles/global.css`** — \~350 lines of wedge styles: header, lookup body, club tabs, clock section, arc/badge, matrix, calibration cards, override grid.

### Key Design Decisions Implemented

- Anchor one position per club → auto-derive all 6 using the ratio table; `derived: true` entries show `est.` badge
- `flightYards` defaults to `yards − 5`; full flight toggle on the matrix
- Interpolation between adjacent positions yields a fractional `~9:30` label and places the clock hand at the correct intermediate angle
- `findBestMatch()` checks both exact nearest and between-position interpolation, returning whichever is closer to target
- All yardages rounded to nearest whole yard

---

### Sequence of Implementation Steps

1. **`src/lib/wedge.ts`** — constants, types, `deriveAllFromAnchor()` (with `flightYards`), `findBestMatch()` with interpolation
2. **`src/hooks/useWedgeCalibration.ts`** — localStorage, expose entries + re-derive helper
3. **`src/components/WedgeClockFace.tsx`** — arc visual, hand, yardage badge, two modes
4. **`src/components/WedgeMatrix.tsx`** — all-clubs table with optional highlight, normal/flight toggle
5. **`src/pages/WedgeCalibratePage.tsx`** — anchor picker + override grid (with flight column)
6. **`src/pages/WedgePage.tsx`** — lookup with clock + matrix + flight variant inline
7. **`src/App.tsx`** — add routes + nav entry point
8. **CSS** — arc layout, yardage badge, matrix table, club tabs, est. indicator

---

## Session 2 — Gauge Polish & Calibration Upgrades

### Gauge Redesign (`WedgeClockFace.tsx`)

The SVG clock face was redesigned into a Ferrari-inspired full-circle instrument:

- **Full bezel ring** with active zone (7:30→Full, left side) in gold and dead zone (right side) in dim white
- **Progress arc** on the bezel fills gold from 7:30 to the selected position
- **Tapered gold needle** rotates to the selected position; rests pointing down at 6 o'clock (the golf ball) when nothing is selected
- **Position dots** on the bezel at each clock position; gold when selected
- **Tap sectors** (pie-slice shaped transparent hit areas) for each position
- **Permanent labels** (yardage + clock time) only on the 4 calibrate positions (7:30, 9:00, 10:30, Full); intermediate positions show nothing on the face — yardage + label appears only in the badge below
- **ZB logo** rendered via JPEG app icon (`zb-logo-new.jpg`) with `clipPath` circular crop and `mix-blend-mode: lighten` to dissolve the dark background
- **Golf ball** at 6 o'clock on the bezel (dead zone): two-ring dimple pattern (center + 6 inner + 9 outer), `wg-ball` radial gradient (off-center for 3D), `wg-ball-rim` shadow gradient for depth
- **Gold needle default**: needle always renders in full gold; rests at 180° (pointing at golf ball) until a position is tapped
- **Selection styling**: gold circle + gold text only for the 4 calibrate positions; non-calibrate positions show no on-face indicator when selected

### Two-Anchor Calibration

- **`src/lib/wedge.ts`**: Added `deriveAllFromTwoAnchors(clubId, pos1, yards1, pos2, yards2)` — fits a line `yards = a·ratio + b` through two measured points for more accurate derivation. `anchors` field changed from `Partial<Record<WedgeClubId, WedgePosition>>` to `Partial<Record<WedgeClubId, WedgePosition[]>>`.
- **`src/hooks/useWedgeCalibration.ts`**: `setAnchor()` accepts optional `second` param; `rederive()` and `migrateCalibration()` use two-anchor math when both anchors stored. Added `normaliseAnchors()` for migrating old string-format anchor storage.
- **`src/pages/WedgeCalibratePage.tsx`**: Added second anchor UI — "Second distance (optional)" position picker + yardage input. Labels updated to "Your yardage" / "Second distance".

### New Clock Positions (8:30, 9:30, 11:30)

Added three half-positions to `WedgePosition`, `CLOCK_POSITIONS`, and `POSITION_ORDER`:

| Key | Label | Ratio | Angle |
| --- | --- | --- | --- |
| `8.5` | 8:30 | 0.70 | −105° |
| `9.5` | 9:30 | 0.80 | −75° |
| `11.5` | 11:30 | 0.98 | −15° |

These are tappable on the gauge (tap sectors cover them) and show yardage in the badge below, but render nothing on the face itself. They are **not** shown in the distance chart.

### Distance Chart (`WedgeMatrix.tsx`)

- New `anchors` prop — cells where the value is a measured anchor show a **gold underline** on the number
- Non-anchor cells render the same-width `wedge-cell-inner` wrapper (dot always present, just invisible) so alignment is unaffected
- 8:30, 9:30, 11:30 filtered out of the chart rows (original 7 positions only)

### Putting Calibration Upgrades (`CalibratePage.tsx`)

- **Two-point linear fit**: added `showPoint2` collapsible section with a second test-distance picker and backstroke adjuster. On save, computes slope `a` and intercept `b` from both points.
- **`distanceOffset`** added to `Calibration` type and threaded through `backswingRaw()`, `getDistanceForBackswingInches()`, `BackswingTable`, `CommonDistances`, and `LookupPage`.
- **Trail Foot** moved back alongside Backstroke in a side-by-side `adjust-area`.
- **"Add a second distance"** disclosure button collapses/expands the second point section.

### Input UX

- `onFocus={e => e.target.select()}` added to all number inputs: wedge calibration anchors, wedge override grid, Mantine `NumberInput` in `DistanceInput` and `SlopeInput`.

### Minor Fixes

- Removed "Back" text from Log and Calibrate page headers (arrow icon only, matching wedge pages)
- Wedge calibrate position buttons made larger (padding and font-size increased)