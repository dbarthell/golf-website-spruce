---
id: GOL-n5qdkq
type: task
title: Few UX clean up tasks
status: Done
---
# Few UX clean up tasks

-Make the Cal button say Calibrate

-Remove logo from Cal page

-Remove redundant Backswing label and value when clicking on a clock position and center the effective uphill/downhill distance and put it on the same line

-Apply superscript/subscript variance correction to clock-mode lateral aim (add superscript inches when putting from above the hole, subtract subscript inches when putting from below)

---

## Implementation Plan

### 1. "Cal" → "Calibrate" button label

**File:** [index.html](code://index.html) · line 60

The `#log-link` anchor currently reads `Cal`. Change the text to `Calibrate`.

```html
<!-- before -->
<a href="" class="full-view-link" id="log-link">Cal</a>

<!-- after -->
<a href="" class="full-view-link" id="log-link">Calibrate</a>
```

> Note: The link text is short enough that it should still fit in the header row alongside the ZeroBreak wordmark + logo. If not, the `full-view-link` padding can be reduced slightly in [mobile.css](code://css/mobile.css).

---

### 2. Remove logo from Cal page

**File:** [log.html](code://log.html)

This change is **already implemented** in the current working copy. The git diff shows the `<div class="header-brand">` wrapper (containing the `<img>` logo and `<h1>`) was already collapsed down to just `<h1>Calibrate</h1>`. No additional code change is needed — we only need to commit the file.

```diff
-    <div class="header-brand">
-      <img src="" alt="" class="header-logo">
-      <h1>Calibrate</h1>
-    </div>
+    <h1>Calibrate</h1>
```

---

### 3. Clock-mode result panel — remove redundant Backswing + compact slope row

**File:** [mobile.js](code://js/mobile.js) · `updateLookupResult()` (clock branch, \~lines 329–386)

#### The problem

When a clock position is selected **and** there is an uphill/downhill slope component, the same adjusted-backswing value (`adjBSDisplay`) is rendered in two places:

1. The main `result-row`: large value + "Backswing" label (left side)
2. The `clockSlopeRowHTML` below it: same value + "backswing" mini-label (right column)

#### The fix

The main `result-row` always shows both **Backswing** and **Aim** — no change there.

The only change is to `clockSlopeRowHTML`: remove the redundant right column (divider + backswing label + backswing value) and center the remaining direction + effective distance:

```js
clockSlopeRowHTML = `
  <div class="slope-row slope-row-centered">
    <span class="slope-dir ${isUphill ? 'slope-up' : 'slope-down'}">${isUphill ? '↑ Uphill' : '↓ Downhill'}</span>
    <span class="slope-dist">${adjDist} ft</span>
  </div>
`;
```

**Add&#32;`.slope-row-centered`&#32;CSS rule** in [mobile.css](code://css/mobile.css) (or reuse an existing rule) to center the contents:

```css
.slope-row-centered {
  justify-content: center;
  gap: var(--space-sm);
}
```

(The existing `.slope-row` already uses `display: flex; align-items: center` so only `justify-content` needs overriding.)

---

### 4. Clock-mode aim — apply superscript/subscript variance correction

**File:** [mobile.js](code://js/mobile.js) · `updateLookupResult()` (clock branch)

#### The problem

The Bryson chart's superscript (+) and subscript (−) values represent inches to **add** (from above the hole) or **subtract** (from below the hole) from the base aim, compensating for the fact that putts played at angles other than 90° to the ZBL spend different amounts of time rolling across the slope.

Currently in clock mode, `zblResult.plusInches` and `zblResult.minusInches` are calculated but silently dropped — only the base aim is used:

```js
const zblAimBase = zblResult ? parseFloat(zblResult.aimInches) : 0;
const lateralAim = Math.round(zblAimBase * breakAbs);
```

#### The fix

`uphillFactor` already encodes whether the putt is from above or below the hole:

- `uphillFactor < 0` → downhill putt → ball from **above** hole → add superscript
- `uphillFactor > 0` → uphill putt → ball from **below** hole → subtract subscript
- `uphillFactor ≈ 0` → pure across (3 or 9 o'clock) → no correction (this is the baseline case)

Scale the correction by `Math.abs(uphillFactor)` so it blends in proportionally — zero at pure 3/9 o'clock (the baseline), growing through the diagonal positions (1, 2, 4, 5 and mirrors). Note: at pure 12/6 o'clock `uphillFactor` is ±1 but `breakAbs` is 0 (straight putt), so the lateral aim — and therefore the correction — is zero there anyway:

```js
const zblAimBase = zblResult ? parseFloat(zblResult.aimInches) : 0;
const plusV  = zblResult ? parseFloat(zblResult.plusInches  || 0) : 0;
const minusV = zblResult ? parseFloat(zblResult.minusInches || 0) : 0;

const aboveBelow = Math.abs(factors.uphillFactor);
const varianceAdj = factors.uphillFactor < 0
  ?  plusV  * aboveBelow   // from above hole: add superscript
  : -minusV * aboveBelow;  // from below hole: subtract subscript

const lateralAim = Math.round((zblAimBase + varianceAdj) * breakAbs);
```

#### Example (10 ft, 3%, clock 2)

- `pct3` at 10 ft = `"13.5 +1.5/−3"` → base 13.5", plusV 1.5", minusV 3"
- Clock 2: `uphillFactor = -cos(60°) = −0.5`, `breakAbs = sin(60°) ≈ 0.866`
- `varianceAdj = 1.5 × 0.5 = +0.75"`
- `lateralAim = round((13.5 + 0.75) × 0.866) = round(12.3) = 12"`
- vs. without correction: `round(13.5 × 0.866) = round(11.7) = 12"` (same after rounding here, but difference grows with larger variance values and steeper angles)

---

### Files changed summary

| File | Change |
| --- | --- |
| `[[index.html](code://index.html)](code://index.html)` | Line 60: `Cal` → `Calibrate` |
| `[[log.html](code://log.html)](code://log.html)` | Already done — logo div removed (working copy only; needs commit) |
| `[[mobile.js](code://js/mobile.js)](code://js/mobile.js)` | `updateLookupResult()` clock branch: remove backswing column from slope row, center direction + distance; apply superscript/subscript variance correction to lateral aim |
| `[[mobile.css](code://css/mobile.css)](code://css/mobile.css)` | Add `.slope-row-centered { justify-content: center; gap: … }` |