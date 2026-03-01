---
id: GOL-64zyyf
type: feature
title: Clock face enhancements
status: Done
---
# Clock face enhancements

When a user clicks on the clock face, I want to show more visual information about how we calculated the ZBL aim in the main row. Some helpful visuals to consider would be ZBL point, variance offset (i.e. superscript/subscript), lateral aim point, target line, actual path the ball will take to the hole, etc. We may need to make the clock a big bigger to show this information. Let me know what you think.

---

## Implementation Plan

### Overview

The plan has two distinct parts:

1. **SVG annotation layer on the clock face** — geometric visuals (target line, ball path, ZBL aim point, variance markers) drawn dynamically when a clock position is selected
2. **Variance display in the result panel** — add the superscript/subscript variance context to the clock-mode "Aim" cell, similar to how normal mode already shows it

The clock face will grow from **160 × 160 px → 200 × 200 px** to give the SVG layer breathing room.

---

### Part 1 — Enlarge the clock face

**File:** [mobile.css](code://css/mobile.css) · `.clock-face`

```css
/* before */
.clock-face {
  width: 160px;
  height: 160px;
}

/* after */
.clock-face {
  width: 200px;
  height: 200px;
}
```

**File:** [mobile.js](code://js/mobile.js) · `initClockFace()`

Update the two layout constants so hour-position buttons track the new size:

```js
// before
const R = 66; // radius from center in px
const C = 80; // center (half of 160px face)

// after
const R = 83; // radius from center in px
const C = 100; // center (half of 200px face)
```

Also update the clock hand height in CSS to match the new radius:

```css
/* before */
.clock-hand { height: 54px; }

/* after */
.clock-hand { height: 68px; }
```

---

### Part 2 — Add the SVG annotation layer

#### 2a. Add the `<svg>` element inside the clock face

**File:** [mobile.js](code://js/mobile.js) · `initClockFace()` — append after existing button loop:

```js
// SVG overlay for visual annotations
const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
svg.id = 'clock-svg';
svg.setAttribute('viewBox', '0 0 200 200');
svg.setAttribute('aria-hidden', 'true');
face.appendChild(svg);
```

**File:** [mobile.css](code://css/mobile.css) — add new rule:

```css
#clock-svg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: visible;
}
```

#### 2b. New `drawClockAnnotations()` function

**File:** [mobile.js](code://js/mobile.js) — add after `initClockFace()`:

```js
function drawClockAnnotations(clockKey, { zblAimBase, lateralAim, plusV, minusV, breakAbs, uphillFactor }) {
  const svg = document.getElementById('clock-svg');
  if (!svg) return;
  svg.innerHTML = '';           // clear previous frame
  if (!clockKey || breakAbs < 0.05) return;   // straight putt — nothing to draw

  const C = 100;  // SVG center (hole)
  const R = 83;   // ball position radius
  const theta = CLOCK_DATA[clockKey].theta;

  // ── Positions ──────────────────────────────────────────────────
  const ballX = C + R * Math.sin(theta);
  const ballY = C - R * Math.cos(theta);

  // Direction from center toward the high side (perpendicular to putt line)
  const highX = -Math.cos(theta);
  const highY = -Math.sin(theta);

  // Visual scale: 1 inch ≈ 1.2 px, capped so large aims stay on-face
  const scale = px => Math.min(px * 1.2, 22);

  const aimPx   = scale(lateralAim);           // final corrected aim
  const basePx  = scale(zblAimBase * breakAbs); // raw ZBL base aim (before variance)

  const aimX  = C + highX * aimPx;
  const aimY  = C + highY * aimPx;
  const baseX = C + highX * basePx;
  const baseY = C + highY * basePx;

  // ── Helper ─────────────────────────────────────────────────────
  function el(tag, attrs) {
    const e = document.createElementNS('http://www.w3.org/2000/svg', tag);
    Object.entries(attrs).forEach(([k, v]) => e.setAttribute(k, v));
    svg.appendChild(e);
    return e;
  }

  // ── 1. Curved ball path (quadratic bezier, ball → hole) ────────
  // Control point bows toward the high side at the midpoint
  const cpX = (ballX + C) / 2 + highX * 18;
  const cpY = (ballY + C) / 2 + highY * 18;
  el('path', {
    d: `M ${ballX} ${ballY} Q ${cpX} ${cpY} ${C} ${C}`,
    stroke: 'rgba(255,255,255,0.55)',
    'stroke-width': '1.5',
    fill: 'none',
    'stroke-linecap': 'round',
  });

  // ── 2. ZBL target line (straight, dashed — ball → aim point) ───
  el('line', {
    x1: ballX, y1: ballY, x2: aimX, y2: aimY,
    stroke: 'rgba(255,255,255,0.80)',
    'stroke-width': '1',
    'stroke-dasharray': '3 3',
    'stroke-linecap': 'round',
  });

  // ── 3. ZBL base aim point (hollow circle) ──────────────────────
  // Only draw when variance correction meaningfully shifts the aim (> 0.5 px)
  if (Math.abs(aimPx - basePx) > 0.5) {
    el('circle', {
      cx: baseX, cy: baseY, r: '2.5',
      stroke: 'rgba(255,255,255,0.60)',
      'stroke-width': '1',
      fill: 'none',
    });
  }

  // ── 4. Final lateral aim point (filled circle) ─────────────────
  el('circle', {
    cx: aimX, cy: aimY, r: '4',
    fill: 'white',
    opacity: '0.90',
  });

  // ── 5. Variance bracket ticks ──────────────────────────────────
  // Perpendicular direction for tick orientation
  const perpX = Math.sin(theta);
  const perpY = Math.cos(theta);
  const tickHalf = 4; // half-length of each tick in px

  function drawTick(offsetPx, color) {
    const tx = C + highX * scale(offsetPx);
    const ty = C + highY * scale(offsetPx);
    el('line', {
      x1: tx + perpX * tickHalf, y1: ty + perpY * tickHalf,
      x2: tx - perpX * tickHalf, y2: ty - perpY * tickHalf,
      stroke: color,
      'stroke-width': '1.5',
      'stroke-linecap': 'round',
    });
  }

  const aboveBelow = Math.abs(uphillFactor);
  if (plusV > 0) {
    const upperAim = (zblAimBase * breakAbs) + plusV * aboveBelow;
    drawTick(upperAim, 'rgba(255,255,255,0.50)');
  }
  if (minusV > 0) {
    const lowerAim = Math.max(0, (zblAimBase * breakAbs) - minusV * aboveBelow);
    drawTick(lowerAim, 'rgba(255,255,255,0.50)');
  }
}
```

#### 2c. Call `drawClockAnnotations()` from the clock branch of `updateLookupResult()`

**File:** [mobile.js](code://js/mobile.js) · `updateLookupResult()` — add one call just before `return`:

```js
// existing line (already present):
result.innerHTML = `...`;

// add immediately after:
drawClockAnnotations(currentClock, {
  zblAimBase,
  lateralAim,
  plusV,
  minusV,
  breakAbs,
  uphillFactor: factors.uphillFactor,
});
return;
```

Also clear the SVG when clock mode is not active. Add to the non-clock branch, just before `result.innerHTML = ...`:

```js
drawClockAnnotations(null, {}); // clear SVG
```

And clear it when the clock is deselected (in the `initClockFace` click handler, toggle-off branch):

```js
if (currentClock === key) {
  currentClock = null;
  ...
  drawClockAnnotations(null, {}); // ← add this
  updateLookupResult();
}
```

---

### Part 3 — Remove variance display from normal-mode result panel

The existing normal-mode "Aim" cell shows `+X" / −Y"` variance context below the ZBL value. Since the clock face SVG annotations will already convey variance visually (via the bracket ticks) whenever a clock position is selected, and the text display takes up space unnecessarily, remove it from the non-clock result panel.

**File:** [mobile.js](code://js/mobile.js) · `updateLookupResult()` non-clock branch — remove the variance block from the Aim cell:

```js
// before
<div class="result-value-container">
  <div class="result-value">${zblDisplay}</div>
  ${hasVariance ? `
    <div class="variance-display">
      ${zblPlusRounded  ? `<span class="variance-plus">+${zblPlusRounded}"</span>`  : ''}
      ${zblMinusRounded ? `<span class="variance-minus">−${zblMinusRounded}"</span>` : ''}
    </div>
  ` : ''}
</div>

// after
<div class="result-value">${zblDisplay}</div>
```

Also remove the `hasVariance`, `zblPlusRounded`, and `zblMinusRounded` variables from the non-clock branch if they are no longer used elsewhere.

---

### Part 4 — Variance superscript/subscript in the clock-mode result panel

Currently the clock-mode "Aim" cell shows only `${lateralAim}"` with no variance context. This change adds it back specifically for the clock branch, so the result panel communicates the range numerically while the SVG annotations show it geometrically.

**File:** [mobile.js](code://js/mobile.js) · `updateLookupResult()` clock branch — replace the Aim `result-item`:

```js
// ── compute variance display for clock mode ──────────────────────
const clockPlusDisplay  = (plusV  > 0 && aboveBelow > 0.05)
  ? Math.round(plusV  * aboveBelow * breakAbs) : null;
const clockMinusDisplay = (minusV > 0 && aboveBelow > 0.05)
  ? Math.round(minusV * aboveBelow * breakAbs) : null;
const hasClockVariance  = clockPlusDisplay || clockMinusDisplay;

// ── Aim cell HTML ────────────────────────────────────────────────
const aimCellHTML = `
  <div class="result-item">
    <div class="result-value-container">
      <div class="result-value">${lateralAim}"</div>
      ${hasClockVariance ? `
        <div class="variance-display">
          ${clockPlusDisplay  ? `<span class="variance-plus">+${clockPlusDisplay}"</span>`  : ''}
          ${clockMinusDisplay ? `<span class="variance-minus">−${clockMinusDisplay}"</span>` : ''}
        </div>` : ''}
    </div>
    <div class="result-label">Aim ${breakDir}</div>
  </div>
`;
```

Then use `aimCellHTML` in the `result.innerHTML` template in place of the current static Aim item.

> **Note:** The displayed variance numbers represent how much the aim would shift at the extremes — they give the user a feel for uncertainty, not a prescription to adjust.

---

### Visual summary of what the user sees

| State | Clock face | Result panel |
| --- | --- | --- |
| No clock position selected | Plain clock (unchanged) | Backswing + plain ZBL aim (no variance display) |
| Clock position selected, flat green | Dashed straight line + filled aim dot | Backswing + `0"` Aim (no variance) |
| Clock position selected, sloped green | Curved ball path + dashed target line + hollow ZBL base dot + filled lateral aim dot + variance ticks | Backswing + corrected aim + `+X" / −Y"` variance |

---

### Files changed summary

| File | Change |
| --- | --- |
| [`[[mobile.css](code://css/mobile.css)](code://css/mobile.css)`](code://css/mobile.css) | Enlarge `.clock-face` to 200×200; update `.clock-hand` height; add `#clock-svg` positioning rule |
| [`[[mobile.js](code://js/mobile.js)](code://js/mobile.js)`](code://js/mobile.js) | Update `R`/`C` constants in `initClockFace()`; append `<svg>` to face; add `drawClockAnnotations()`; call it from the clock branch of `updateLookupResult()`; remove variance display from non-clock Aim cell; add variance display to clock-mode Aim cell |