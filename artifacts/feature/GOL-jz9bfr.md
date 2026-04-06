---
id: GOL-jz9bfr
type: feature
title: New module for club yardages
status: In Review
---
# New module for club yardages

I want to build one more module for hold the rest of a user's yardages (e.g. PW - Driver). I want to primarily do this so I can create a widget from the lock screen on my iPhone for easy reference while I play but I also want the UI to look cool too like the putting and wedges modules. We may need to introduce a bottom nav bar now that the top is getting crowded with icons. So the nav bar would have icons for ZeroBreak (Putting), Wedges, and [Name] for new module. Other things I care about:

1. Carry and total distance for full shot as well as carry and total distance for what Phil Mickelson calls “Pelz shot” though we shouldn’t call it that: "[philmickelson](https://www.instagram.com/philmickelson/)

 279w

Building a reference. 
This is my practice session tonight working on my “Pelz” 9 iron. A Pelz shot is a shorter backswing to take 10 yards off my “stock” shot. My stock 9 iron goes 155 and my Pelz 9 iron goes 145. It also flies lower and has less spin than my full or or stock shot. This shot is critical for distance control in the wind, controlling spin, getting to back pins, and to not have any distance gaps between my irons. It wasn’t until 2004 that I worked hard on this shot and I wasn’t truly a great iron player until I had a consistent reference like this. (These shots were all consecutive). I like to have the slightest of fade because by doing this, the face isn’t rotating as fast through impact and leads to more consistent distance control. I hope this helps with understanding how detailed the practice sessions are for the guys on Tour and how precise the guys are with their iron play.” This is only for irons since you don’t really shorten your backswing for woods or driver. 

2. Some way for a user to customize their set bc some have hybrids, 5W, 7W, mini driver, etc.
3. Let’s make it manual inputs for now since we’re not integrated with a launch monitor technology and we don’t want to extrapolate for all irons like we do wedges or putting strokes. However, we can extrapolate for their Pelz/10:30 yardages since most users won’t have enough practice time to dial those in. But should also have a way to manually configure those too if desired. 
4. I’m still unsure what the UI should look like for this module but the iPhone widget should be easy to glance at and look up any number for any iron, club or driver with miminal scrolling or button pushing

---

## Implementation Plan

### Decisions Logged

| # | Decision |
| --- | --- |
| 1 | Module is named **"My Clubs"**. The page shows "My Clubs" as a large top-left header (matching the "Wedges" header style). The bottom tab bar is **icon-only** — no labels. |
| 2 | **Total distance is the primary field.** A club only appears in the reference table once at least a full-swing total is entered. Carry is optional and shown when provided. |
| 3 | **The global Pelz offset is a rough starting estimate only.** Each player's 10:30 window is unique to them and varies by club — "your clock is not going to match up to my clock." The global offset (default 10 yds) pre-fills auto-extrapolated values so a user has something on day one, but the UI will clearly label these as estimates and make per-club manual calibration the encouraged path. Once a user measures and enters their actual 10:30 yardage for a club, the estimate is discarded. |
| 4 | **Wedges are kept fully separate.** GW/SW/LW are regular clubs in My Clubs with their own independently-entered full and 10:30 yardages. The Wedges module uses a different swing (narrower stance, different tempo) so those clock-position yardages don't translate — no data sharing between modules. |
| 5 | **Immediate nav cutover.** Wedges hasn’t launched yet, so the top-header Wedges link is removed outright. Back arrow on WedgePage removed. All inter-module navigation moves to the bottom tab bar. |
| 6 | **User-configurable widget club selection** via WidgetKit Intent. The medium/large widget will attempt to show all clubs (scrolling or two-column layout if supported by WidgetKit constraints). |
| 7 | **Yards only** for this module. |

---

### Summary

We’re building the **"My Clubs"** module — a full-bag yardage reference (Driver through wedges) with a clean, glanceable table showing total and carry distances for both full swing and 10:30 shots. It slots in as the third module alongside ZeroBreak (putting) and Wedges, connected via a new icon-only bottom tab bar that replaces the increasingly crowded top-header navigation. The iPhone lock screen widget requires a native WidgetKit Swift extension.

---

### What Needs to Change and Why

The app has two modules linked from the top header. A third module tips the header over its comfortable limit, and bottom-tab navigation is explicitly desired. My Clubs has a fundamentally different data shape from wedges (per-club total/carry for a customizable set) so it gets its own fully independent data layer. GW/SW/LW appear in My Clubs as regular clubs with manually-entered yardages — the Wedges module’s clock-position data is not used here because that swing (narrower stance, different tempo) produces different yardages than a player’s "full/10:30" reference swing.

---

### New Files to Create

| File | Purpose |
| --- | --- |
| [src/lib/bag.ts](code://src/lib/bag.ts) | `BagClub`, `BagEntry`, `BagData`, `BagSettings` types; default club list; Pelz extrapolation helper |
| [src/hooks/useBagData.ts](code://src/hooks/useBagData.ts) | localStorage R/W, CRUD for clubs + entries, exposes computed Pelz values |
| [src/pages/BagPage.tsx](code://src/pages/BagPage.tsx) | Main read-only reference view — the "on course" screen |
| [src/pages/BagEditPage.tsx](code://src/pages/BagEditPage.tsx) | Input/calibration page — enter totals, optional carry, override 10:30, configure Pelz offset |
| [src/components/BagTable.tsx](code://src/components/BagTable.tsx) | Compact yardage table (reusable — used in BagPage and eventually the widget preview) |
| [src/components/BottomNav.tsx](code://src/components/BottomNav.tsx) | Icon-only fixed bottom tab bar (3 tabs) |
| `ios/App/App/BagWidget/BagWidget.swift` | WidgetKit extension — lock screen + home screen sizes |
| `ios/App/App/BagWidget/BagEntry.swift` | Shared data model for Swift widget to decode from App Group UserDefaults |

---

### Files to Modify

| File | Change |
| --- | --- |
| [src/App.tsx](code://src/App.tsx) | Add `/bag` and `/bag/edit` routes; render `<BottomNav>` outside `<Routes>` so it persists across all pages |
| [src/pages/LookupPage.tsx](code://src/pages/LookupPage.tsx) | Remove Wedges link from top header; remove Log + Calibrate links too if those move (see note below) |
| [src/pages/WedgePage.tsx](code://src/pages/WedgePage.tsx) | Remove back-to-putting arrow; "Wedges" becomes the large top-left module title |
| [src/styles/global.css](code://src/styles/global.css) | Add BottomNav styles; bump body `padding-bottom` to ~56px + `env(safe-area-inset-bottom)` |
| `ios/App/App.xcodeproj` | Add BagWidget target; add App Group entitlement to both main target and widget target |

> **Note on Log + Calibrate links:** These currently live in the LookupPage top header alongside the Wedges link. With bottom nav taking over module switching, those secondary actions should either stay in the top header (now less crowded) or move to a gear/settings icon. Recommend keeping them in the top header for now — they’re page-specific actions, not module navigation.

---

### Data Model (`src/lib/bag.ts`)

```ts
export type ClubCategory = ‘driver’ | ‘wood’ | ‘hybrid’ | ‘iron’ | ‘wedge’;

export interface BagClub {
  id: string;          // ‘dr’, ‘3w’, ‘4h’, ‘7i’, ‘pw’, etc.
  label: string;       // ‘Driver’, ‘3W’, ‘4H’, ‘7i’, ‘PW’
  category: ClubCategory;
  order: number;       // ascending = longer club first (Driver = 0)
  hasPelz: boolean;    // true for irons + wedges; false for driver/woods
}

export interface BagEntry {
  clubId: string;
  fullTotal: number;         // REQUIRED — gate for showing the club in the table
  fullCarry: number | null;  // optional
  pelzTotal: number | null;  // null = auto-extrapolate from fullTotal
  pelzCarry: number | null;  // null = auto-extrapolate from fullCarry (if present)
  pelzOverride: boolean;     // true = user manually set pelz values
}

export interface BagSettings {
  pelzOffset: number;        // default 10 — subtracted from full totals/carries to estimate 10:30
}

export interface BagData {
  clubs: BagClub[];
  entries: BagEntry[];
  settings: BagSettings;
}
```

**10:30 yardage philosophy:** The 10:30 swing is not a simple formula — it is "pretty much your full swing, trusting the length, not adding power." The relationship between a player's full swing and 10:30 window is unique per player per club. The global offset produces a placeholder only. Once a user has actually hit balls at 10:30 and knows their number, they enter it and it replaces the estimate entirely.

**Pelz computation** (computed on read, never stored):

```ts
function computedPelzTotal(entry: BagEntry, offset: number): number {
  return entry.pelzOverride ? entry.pelzTotal! : entry.fullTotal - offset;
}
function computedPelzCarry(entry: BagEntry, offset: number): number | null {
  if (entry.pelzOverride) return entry.pelzCarry;
  return entry.fullCarry !== null ? entry.fullCarry - offset : null;
}

// Is this value a real measured number, or just an estimate?
function isPelzCalibrated(entry: BagEntry): boolean {
  return entry.pelzOverride;
}
```

The `isPelzCalibrated` flag is used in the reference table to visually distinguish measured 10:30 values from estimated ones (e.g. a small dot indicator, similar to how the Wedges module marks anchor positions in gold).

**Default club set** (in order): Driver · 3W · 5W · 3H · 4H · 4i · 5i · 6i · 7i · 8i · 9i · PW · GW · SW · LW

GW/SW/LW are regular `BagClub` entries with `hasPelz: true`, entered and managed entirely within My Clubs. They have no connection to the Wedges module — a player’s full-swing GW yardage from a normal address is a distinct number from their wedge clock yardages.

---

### Bottom Nav — Icon-Only (`src/components/BottomNav.tsx`)

- Fixed to bottom of viewport; `env(safe-area-inset-bottom)` padding on iPhone
- Dark green background (`--green-dark`), \~56px tall above the safe area
- Active icon highlighted in gold (`--green-accent`)
- Three icons (all from `@tabler/icons-react`):
- **Putting:** `IconCircleDot` or `IconTargetArrow`
- **Wedges:** `IconGauge` (already used in the header today)
- **My Clubs:** `IconCards` or `IconLayoutList` — to be confirmed during implementation
- Tapping the active tab scrolls the current page to top (no re-render)
- Sub-pages (`/wedges/calibrate`, `/bag/edit`) highlight their parent tab

---

### BagPage — Main Reference View

#### Design reference: Arccos "Club Distances"

Arccos uses horizontal bars scaled to distance — one number per club, three swipeable views (Longest / Smart / Range). The bar approach is genuinely valuable because **distance gaps become visible at a glance** (you can immediately see when 7i and 8i are too close). Their layout works because they only show one number per club per screen.

We have two shot types (full + 10:30) which rules out a straight copy — but the bar idea is worth adapting. Two directions to consider:

**Option A — Swipeable views (Arccos-style):**

- "Full" view: one bar + total per club, very clean and scannable
- "10:30" view: same layout for the 10:30 numbers
- Carry shown as secondary text below the bar or on tap
- Pro: maximally glanceable, each screen is simple; Con: you can't compare full vs. 10:30 side-by-side

**Option B — Dual-bar rows (our own thing):**

- Each club row has two stacked bars — a longer one for full, a shorter one for 10:30 — with the numbers on the right
- Makes the relationship between full and 10:30 visible in the same row
- Carry is secondary (small text below or on tap)
- Pro: full picture in one scroll; Con: slightly more complex per row

**Recommendation:** Option A for the primary reference view (cleaner for on-course), with the view toggle prominently placed at the top rather than hidden in a swipe (the Wedge module's Normal/Flight toggle is a good precedent for this). Option B could be explored as a "gaps view" later.

#### Layout

```
┌─────────────────────────────────────────┐
│  My Clubs              [edit icon]       │  ← dark green header
├─────────────────────────────────────────┤
│  [ Full ]  [ 10:30 ]                    │  ← toggle (like wedge Normal/Flight)
├─────────────────────────────────────────┤
│  DR  ████████████████████████  295      │
│  3W  █████████████████████     260      │
│  5W  ████████████████████      240      │
│  4H  ██████████████████        220      │
│  5i  █████████████████         205      │
│  6i  ████████████████          190      │
│  7i  ███████████████           178      │
│  8i  ██████████████            163      │
│  9i  █████████████             150      │
│  PW  ████████████              137      │
│  GW  ███████                   115      │
│  SW  █████                      98      │
│  LW  ████                       80      │
└─────────────────────────────────────────┘
```

- **Bars** scale proportionally to the longest club (Driver = 100% width); gives immediate gap visibility
- **Number on the right** is the total distance — large and easy to read
- **Carry** shown as smaller secondary text directly below the total when entered (e.g. `carry 275` in muted text)
- **10:30 toggle** switches every bar and number to 10:30 values; bar widths re-scale to the longest 10:30 club
- Estimated 10:30 values (not yet measured) shown with `~` prefix and muted bar color; measured values shown in gold/bright
- Tap a row to highlight/pin it (gold left-border) for on-course thumb-reference
- `hasPelz: false` clubs (Driver, woods) show `—` and no bar in 10:30 view
- Empty state: CTA card to edit page
- Edit icon (top-right of header) navigates to `/bag/edit`

---

### BagEditPage — Input / Calibration

**Global setting (top of page):**

```
  10:30 default estimate   [ 10 ] yds less than full
  Used only for clubs you haven't measured yet — your actual 10:30
  window is unique to you. Enter real numbers below when you have them.
```

**Per-club rows:**

```
  7i    Full  Total [ 205 ]  Carry [ 195 ]
        10:30 Total [ 195~ ] Carry [ 185~ ]   ← estimated, tap to enter real number
```

```
  7i    Full  Total [ 205 ]  Carry [ 195 ]
        10:30 Total [ 193  ] Carry [ 183  ]  ✓ ← measured, locked in
```

- `fullTotal` input is the primary field; required before the club appears in the reference table
- `fullCarry` input is optional; shown dimmed with placeholder if blank
- 10:30 row: auto-estimated values shown with a `~` tilde and a muted color + small "est." label, clearly communicating this is not a measured number. Tapping unlocks the input so the user can enter their real yardage. Once entered, the `~` and muted style disappear — replaced with a small `✓` or gold dot (matching wedge anchor indicator style) to show it's been calibrated.
- The section header above the 10:30 row has a brief inline note: *"10:30 swing = full swing, trusting distance not power. Your number, not a formula."*
- 10:30 inputs hidden entirely for Driver / woods (`hasPelz: false`)
- **Add club** button opens a bottom sheet with standard extras (2i · 3i · 5W · 7W · Mini Driver · Custom...)
- Remove: trash icon on each row, tap to confirm
- Saves on blur; no explicit save button (same pattern as [WedgeCalibratePage.tsx](code://src/pages/WedgeCalibratePage.tsx))

---

### iOS Lock Screen Widget

Requires a native **WidgetKit** Swift extension — Capacitor/web alone cannot power lock screen widgets.

**Data bridge:**

- Add App Group `group.com.zerobreak.shared` to both the main app target and the widget target
- `useBagData` writes a JSON snapshot to `UserDefaults(suiteName: "group.com.zerobreak.shared")` every time bag data changes (via a thin Capacitor plugin call or `@capacitor/preferences` pointed at the group container)
- The Swift widget decodes this snapshot; no network call required

**Widget sizes & content:**

| Size | Content |
| --- | --- |
| `accessoryRectangular` (lock screen) | User-selected clubs (via Intent), total yardage only, 4–6 rows |
| `systemSmall` | One club: Full total + 10:30 total |
| `systemMedium` | Full bag two-column layout (all clubs, total only) — attempts to show everything |
| `systemLarge` | Full bag with both total + carry columns |

**WidgetKit Intent configuration:** A `ConfigurationIntent` lets the user choose which clubs show on the lock screen widget. For medium/large sizes, all clubs are shown automatically (no configuration needed — just display the full list).

The Swift extension is self-contained and ships in a follow-up PR once the web data model is stable and tested.

---

### Implementation Sequence

1. **Data layer** — `src/lib/bag.ts` + `src/hooks/useBagData.ts` (types, default clubs, localStorage R/W, Pelz computation, Pelz offset setting)
2. **BagEditPage** — inputs, add/remove clubs, global Pelz offset, save on blur
3. **BagPage** — reference table, tap-to-highlight, empty state
4. **BottomNav** — icon-only tab bar + wire routes in App.tsx + CSS `padding-bottom` adjustments
5. **Header cleanup** — remove Wedges link from LookupPage header; remove back arrow from WedgePage; add "My Clubs" and "Wedges" as large top-left module titles
6. **iOS widget** — App Group entitlement, shared UserDefaults write bridge, Swift WidgetKit extension with Intent config *(follow-up PR)*

Steps 1–5 are pure web/React and ship as one PR. Step 6 is iOS-native and ships separately.

---

## Implementation Notes (Steps 1–5 Complete)

### Files Created

- [bag.ts](code://src/lib/bag.ts) — `BagClub`, `BagEntry`, `BagData`, `BagSettings` types; `DEFAULT_CLUBS` (DR · 3W · 5W · 3H · 4H · 4–9i · PW · GW · SW · LW); `EXTRA_CLUBS` for the add sheet; `computedPelzTotal`, `computedPelzCarry`, `isPelzCalibrated`, `emptyBagData`
- [useBagData.ts](code://src/hooks/useBagData.ts) — localStorage R/W, `setEntry`, `clearEntry`, `addClub`, `removeClub`, `updateSettings`, `getEntry`; exposes `allClubs` (sorted) and `calibratedClubs` (entries-gated)
- [BagPage.tsx](code://src/pages/BagPage.tsx) — Reference view with Full / 10:30 toggle; proportional horizontal bars scaled to longest club; carry as secondary text; estimated 10:30 values shown with `~` prefix and muted bar; tap-to-pin row (gold left border); empty state CTA
- [BagEditPage.tsx](code://src/pages/BagEditPage.tsx) — Global pelzOffset card; per-club rows with Full Total/Carry + 10:30 section; estimated 10:30 shown as a tappable "unlock" button that pre-fills the estimate; calibrated rows show gold dot + checkmark; save-on-blur; remove with confirm; Add Club bottom sheet with predefined extras + custom input
- [BottomNav.tsx](code://src/components/BottomNav.tsx) — Icon-only fixed bottom tab bar (Putting · Wedges · My Clubs); active tab in gold; tapping active tab scrolls to top; sub-pages highlight parent tab

### Files Modified

- [App.tsx](code://src/App.tsx) — Added `/bag` and `/bag/edit` routes; `<BottomNav />` rendered outside `<Routes>` so it persists across all pages
- [LookupPage.tsx](code://src/pages/LookupPage.tsx) — Removed Wedges link from top header (now in bottom nav)
- [WedgePage.tsx](code://src/pages/WedgePage.tsx) — Removed back arrow; header now uses `bag-header` / `bag-module-title` style (large left-aligned module title, matching My Clubs)
- [global.css](code://src/styles/global.css) — Body `padding-bottom` bumped to include 56px nav height + safe area; added all bottom nav, My Clubs reference view, and edit page styles

### Architecture notes

- State is lifted in `BagEditPage` — all `setEntry`/`clearEntry` calls go through a single `useBagData()` instance passed as props to `ClubRow`, avoiding multi-instance localStorage overwrite bugs
- 10:30 values are computed on read in `BagPage` via `computedPelzTotal/Carry`; never stored as derived values
- GW/SW/LW are fully independent entries in My Clubs — no data sharing with the Wedges module

### Still to do (Step 6 — follow-up PR)

- iOS App Group entitlement (`group.com.zerobreak.shared`)
- `useBagData` → UserDefaults JSON bridge via Capacitor
- `BagWidget.swift` WidgetKit extension (lock screen + home screen sizes)
- `BagEntry.swift` shared data model