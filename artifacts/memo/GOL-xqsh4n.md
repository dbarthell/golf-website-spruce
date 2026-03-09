---
id: GOL-xqsh4n
type: memo
title: New features and enhancements
---
# New features and enhancements

What other features or enhancements would you add to the app?

---

## Current App Summary

ZeroBreak is a putting calculator built around two complementary methods:

1. **Backswing control** — from a lag putting table, mapped to trail-foot body landmarks (inside foot, middle, outside foot). A personal `distanceFactor` calibration scales all distances to the user's actual stroke length.
2. **ZBL aim** — the Zero Break Line: the straight-line aim point above the hole that lets gravity curve the ball in. Uses the Bryson slope table, interpolated by distance (1–50+ ft) and slope (1–6%), then scaled by Stimp.

The UI is a single-page lookup tool (distance + slope + stimp + clock position → backswing + aim) with a Calibrate page for one-time stroke calibration. State is persisted in `localStorage`. The app ships as a PWA with a service worker and web manifest.

---

## Feature Ideas & Enhancements

### 1. 🎯 Show ZBL Variance (Quick Win)

**What:** The `calculateZBLVector` function already returns `plusInches` / `minusInches` variance values from the Bryson table, but they are never surfaced to the user in the main result panel. In clock mode, only `zblAimBase` is used; the variance terms drop out silently.

**Why it matters:** Variance tells you how much the aim window expands or contracts depending on green reading uncertainty. At 30 ft / 4% slope, the band might be ±1.5". This is actionable info.

**What to change:**

- `src/components/LookupResult.tsx` — in `straight` mode, add a small `±` line under the ZBL aim value when `plusInches`/`minusInches` are non-null
- `src/pages/LookupPage.tsx` — thread `plusInches`/`minusInches` through `LookupResult` type (or add to `annotations`)
- `src/lib/types.ts` — extend `LookupResult` straight/clock variants with optional `zblPlus` and `zblMinus` fields

**Effort:** Small — mostly plumbing from existing data to UI.

---

### 2. 📋 Round Putt Log

**What:** A lightweight session log that lets golfers record each putt (distance, slope, clock, outcome: made / short / long / left / right) during a round and review tendencies at the end.

**Why it matters:** Most golfers have no idea how they actually putt. A simple log turns ZeroBreak from a lookup tool into a practice and analytics companion.

**What to change:**

- New page `src/pages/LogPage.tsx`
- New hook `src/hooks/usePuttLog.ts` — persists to `localStorage` as a JSON array, with a `roundId` (date-based) so data doesn't bleed across sessions
- New `App.tsx` route `/log`
- Header link on `LookupPage` alongside the Calibrate icon
- Result panel gets a "Log this putt" button (opens a small inline outcome picker — made / miss direction)
- Log page shows a summary: putts by distance bucket, miss tendency, average distance vs. table

**Key data shape:**

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

**Effort:** Medium. Pure client-side. No backend needed.

---

### 3. 💾 Multiple Calibration Profiles

**What:** Right now there is a single `distanceFactor` in `localStorage`. Support named profiles (e.g., "Home Course", "Range Practice", "Fast Bermuda") so golfers can switch contexts without re-calibrating every time.

**Why it matters:** Green speed and personal stroke feel vary meaningfully between home course, away events, and practice greens. Switching profiles should be instant.

**What to change:**

- `src/hooks/useCalibration.ts` — extend storage key to hold a `{ profiles: Record<string, Calibration>, active: string }` shape; default profile named `"default"`
- `src/pages/CalibratePage.tsx` — add a profile selector row at the top (chips or a `<select>`); "New Profile" button opens a name input
- `src/lib/types.ts` — add `CalibrationProfile` type

**Backwards compatibility:** If the old `putt-cal` key exists in `localStorage`, migrate it to a `"default"` profile on first load.

**Effort:** Medium — mostly state management and a small UI addition to Calibrate page.

---

### 4. 🔄 Distance Units Toggle (Feet ↔ Metres)

**What:** A setting to display distances in metres instead of feet. The underlying data and calculations stay in feet; only display values are converted.

**Why it matters:** Golfers outside the US measure in metres. The ZBL aim output (in inches) stays as-is; only distance inputs and backswing-table distances change.

**What to change:**

- New `src/hooks/useUnits.ts` — persists `'ft' | 'm'` to `localStorage`
- `src/components/DistanceInput.tsx` — show "ft" vs "m" label; convert on input/output
- `src/components/CommonDistances.tsx` — convert display labels
- `src/components/BackswingTable.tsx` — convert distance column
- `src/pages/CalibratePage.tsx` — convert test-distance chips and preview table
- Global toggle button in header

**Conversion:** `1 ft = 0.3048 m`. Input in metres is converted to feet before any calculation; outputs in feet are converted to metres for display only.

**Effort:** Small-medium. Most complexity is threading the `units` value to every component that shows a distance.

---

### 5. 📱 Improved PWA Install Experience

**What:** The app already ships with [manifest.json](code://manifest.json) and [sw.js](code://sw.js), but there is no in-app prompt to encourage installation. Add a dismissible banner on the main page that appears when the browser `beforeinstallprompt` event fires.

**Why it matters:** Most users never find the browser's native "Add to Home Screen" option. An explicit in-app prompt dramatically increases install rates. A PWA putt tool on the home screen is much faster to open on the course.

**What to change:**

- New `src/hooks/useInstallPrompt.ts` — captures the `beforeinstallprompt` event and exposes `{ canInstall, prompt, dismiss }`
- New `src/components/InstallBanner.tsx` — a slim dismissible bar at the top of `LookupPage` (shown only when `canInstall && !dismissed`)
- Dismissed state saved to `localStorage` so it doesn't re-appear each visit

**Effort:** Small — mostly a hook that listens for a browser event.

---

### 6. ⚡ "Quick Putt" Haptic Feedback (Mobile)

**What:** On mobile, trigger a short haptic pulse (`navigator.vibrate`) when:

- The result updates (new distance entered)
- The user taps a clock position
- Calibration is saved

**Why it matters:** On-course, golfers often glance at the screen and don't want to read a confirmation. A tactile cue confirms the input registered without requiring audio.

**What to change:**

- New `src/lib/haptics.ts` — thin wrapper: `vibrate(pattern: number | number[])` with a feature-detect guard
- Call sites in `LookupPage.tsx` (result change via `useEffect`), `ClockFace.tsx` (clock button onClick), `CalibratePage.tsx` (save button)

**Effort:** Tiny.

---

### 7. 📏 Physical Ruler Export — "ZeroBreak Calibration Kit"

**Context:** There is a planned physical product — custom laser-engraved steel/aluminum rulers that pair with the app. The business model is: *calibrate on the app → app generates your personal stroke zones → buy a custom ruler with those zones pre-marked*. This feature bridges the digital calibration to the physical tool.

**What:** A "Print / Export Your Ruler" screen on the Calibrate page that generates a personalized ruler specification based on the user's `distanceFactor` and preferred Stimp. The output has two forms:

1. **Printable PDF** — A 1:1 scale paper ruler the user can print at home and use immediately as a "Starter Ruler" while they wait for the physical product. Includes their stroke-zone marks, foot landmarks, and a QR code back to the app.
2. **Order Summary Card** — A shareable spec sheet (PNG or PDF) showing their personal "Zone Map" (e.g., *10 ft = 4.5"*, *20 ft = 8.25"*, etc.) that they attach to a purchase order for a custom-engraved ruler.

**The Zone Map logic** (all data already in the app):

For each common distance (5, 6, 10, 15, 20, 25, 30, 40, 50 ft), at the user's saved Stimp, compute `backswingDisplay(dist, stimp, lagRows, distanceFactor)`. This is the inch mark that gets laser-etched on their personal ruler. The app already knows these exact numbers — the export just formats them for print/manufacturing.

**What to change:**

- New page `src/pages/RulerPage.tsx` — displays the Zone Map table and generates the export
- New `App.tsx` route `/ruler`
- Navigation link on Calibrate page: "Get Your Ruler →"
- New `src/lib/rulerExport.ts` — builds the zone map array from calibration + lag data; renders to `<canvas>` or uses a library like `jsPDF` for PDF output
- The printable ruler should be formatted at exactly **1 inch = 1 inch** at 96 DPI so it prints correctly from a standard browser print dialog

**Key design decisions:**

- The ruler should show the **trail-foot landmarks** (inside foot ≈ 9", middle ≈ 12", outside foot ≈ 18") as named zones rather than raw inch values — matching the body-landmark system the app already teaches
- Include a QR code (use a lightweight client-side QR library, e.g., `qrcode`) that deep-links to `/calibrate` so the physical product always connects back to the app

**Business model hook:**

- Free tier: printable paper ruler
- Paid tier / upsell: "Order a Custom Engraved Ruler" button links to an order form (Shopify, Gumroad, or simple email form to start) pre-populated with the user's Zone Map spec

**Effort:** Medium-large. The zone map calculation is trivial (reuses existing lib functions). The main work is the PDF/canvas rendering and print layout.

---

## Proposed Breakdown

1. [GOL-hpt7mx](artifact://GOL-hpt7mx) — Thin `haptics.ts` wrapper around `navigator.vibrate`; fire on result update, clock tap, and calibration save. *(Size: S)*
2. [GOL-zblv4r](artifact://GOL-zblv4r) — Surface the existing `plusInches`/`minusInches` from `calculateZBLVector` as a `±` line in the straight-putt result panel. *(Size: S)*
3. [GOL-pwa2ib](artifact://GOL-pwa2ib) — `useInstallPrompt` hook + dismissible `InstallBanner` component triggered by `beforeinstallprompt`. *(Size: S)*
4. [GOL-unit5t](artifact://GOL-unit5t) — `useUnits` hook + conversion threaded through all distance/aim display components; aim converts to cm in metric mode. *(Size: M)*
5. [GOL-cal3pr](artifact://GOL-cal3pr) — Extend `useCalibration` to named profiles with migration from existing `putt-cal` key; profile selector UI on Calibrate page. *(Size: M)*
6. [GOL-log8rp](artifact://GOL-log8rp) — `usePuttLog` hook + `LogPage` + inline outcome picker on result panel; free tier shows last 3 rounds with premium history upsell stub. *(Size: M)*
7. [GOL-rul9ex](artifact://GOL-rul9ex) — `RulerPage` + `rulerExport.ts` generating a personalized SVG ruler (1:1 print, DXF-compatible) with QR code and physical product order CTA. *(Size: L)*

---

## Open Questions for Human Review

1. **Putt log scope** — Should the log persist across rounds indefinitely (with round separation), or should it be reset per session? Infinite history adds complexity to the UI and storage budget.

- **Decision:** Implement infinite history with round separation as a **premium feature**. Free tier shows the last 3 rounds; premium unlocks full history and CSV/JSON export. This creates a natural upsell anchor.

2. **Calibration profiles** — Is the UX of "named profiles" the right shape, or would a simpler "quick adjust" slider on the main page suffice (without the full calibrate flow)?

- **Decision:** Named profiles is the right UX for users who play multiple courses. A lightweight "quick adjust" nudge (±10%) on the main page is a good complementary feature for on-the-fly tuning without replacing the full Calibrate flow.

3. **Units toggle** — Global setting or per-session? Should the Bryson aim output (inches) also convert to centimetres in metric mode?

- **Decision:** Global setting, persisted to `localStorage`. Yes — aim output (inches) should convert to centimetres in metric mode for consistency. Metric users shouldn't have to mentally convert.

4. **ZBL variance display** — Should the variance band appear as text (e.g., `3" ±1"`) or as a visual range indicator on the clock SVG?

- **Decision:** Ship inline text first (`3" ±1"` beneath the aim value) — cleaner, faster to implement. Reserve the clock SVG visual band for a later polish pass once we validate users find the variance useful.

5. **Ruler export format** — PDF via `jsPDF`, plain `<canvas>` download, or SVG (easiest to hand off to a laser engraver as a DXF-compatible vector)? Should the "paper ruler" be standard 8.5×11 (prints at \~24") or letter-size tiled for a full 48"?

- **Decision:** SVG is the strongest format — vector, DXF-compatible for laser engravers, and infinitely scalable. For the printable paper ruler, generate 8.5×11 landscape pages (tiled if the ruler exceeds one page). SVG → browser print dialog keeps dependencies minimal.

6. **Ruler order flow** — What is the fulfillment path for the physical product? Shopify storefront, manual email order, or a third-party print-on-demand shop (e.g., SendCutSend API)?

- **Decision:** Not determined yet. Will revisit when the product roadmap and first manufacturing path are clearer.

7. **Priority order** — Which of these would be most useful to tackle first?

- **Decision:** Not determined yet. Recommend sequencing by effort/impact: #6 Haptic Feedback (tiny), #1 ZBL Variance (small), #5 PWA Install / App Store path (small-medium), then #4 Units Toggle, #3 Calibration Profiles, #2 Putt Log, #7 Ruler Export.