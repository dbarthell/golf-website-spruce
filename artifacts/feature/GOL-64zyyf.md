---
id: GOL-64zyyf
type: feature
title: Clock face enhancements
status: Shipped
---
# Clock face enhancements

When a user clicks on the clock face, I want to show more visual information about how we calculated the ZBL aim in the main row. Some helpful visuals to consider would be ZBL point, variance offset (i.e. superscript/subscript), lateral aim point, target line, actual path the ball will take to the hole, etc. We may need to make the clock a big bigger to show this information. Let me know what you think.

---

## What was built

### Clock face SVG annotations

An SVG overlay (`#clock-svg`) sits inside the clock face and draws geometric visuals based on distance, slope, and clock position. The SVG scales automatically with the CSS clock size via `viewBox="0 0 200 200"` and `width/height: 100%`.

**Always visible** (as soon as distance + slope are entered):

- Thin vertical measurement line from cup center → ZBL dot
- Filled white ZBL dot at `(C, C − zblPx)` — fixed at 12 o'clock above the cup
- `ZBL` label above the dot; inch value alongside the measurement line

**When a clock position is selected with break:**

- Dashed line from ball position → ZBL dot (the zero break line)

**Removed from SVG (considered and rejected):**

- Curved ball path to hole
- Lateral aim line/dot/diamond
- Triangle hypotenuse
- Variance bracket ticks

The lateral aim is instead communicated in the result panel (see below).

---

### Clock face sizing

Clock face enlarged to **280 × 280 px** to give the SVG layer breathing room. Hour-position button layout constants updated to match.

[mobile.css](code://css/mobile.css)

```css
.clock-face {
  width: 280px;
  height: 280px;
}

.clock-hand { height: 95px; }

.clock-pos {
  width: 38px;
  height: 38px;
  font-size: 0.85rem;
}

#clock-svg {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  pointer-events: none;
  overflow: visible;
}
```

[mobile.js](code://js/mobile.js)**&#32;·&#32;`initClockFace()`**

```js
const R = 116; // radius from center in px
const C = 140; // center (half of 280px face)
```

---

### `drawClockAnnotations()` — final design

```js
function drawClockAnnotations(clockKey, { zblAimBase = 0, lateralAim = 0, breakAbs = 0 } = {}) {
  const svg = document.getElementById('clock-svg');
  if (!svg) return;
  svg.innerHTML = '';
  if (!zblAimBase || zblAimBase <= 0) return;

  const C = 100;  // SVG center (hole)
  const R = 83;   // ball position radius
  const scaleIn = inches => Math.min(Math.max(inches * 3, 10), 45);

  // ZBL: always directly above cup at 12 o'clock
  const zblPx = scaleIn(zblAimBase);
  const zblX  = C;
  const zblY  = C - zblPx;

  // Dashed ball→ZBL line only when clock position with break is active
  const hasBreak  = clockKey && breakAbs >= 0.05;
  const theta     = hasBreak ? CLOCK_DATA[clockKey].theta : null;
  const breakSign = hasBreak ? (Math.sin(theta) >= 0 ? 1 : -1) : 1;

  function el(tag, attrs) { /* createElement helper */ }

  // 1. Measurement line: cup → ZBL (always)
  el('line', { x1: C, y1: C, x2: C, y2: zblY, stroke: 'rgba(255,255,255,0.40)', 'stroke-width': '1' });

  // 2. Dashed ball→ZBL line (only when clock selected)
  if (hasBreak) {
    const ballX = C + R * Math.sin(theta);
    const ballY = C - R * Math.cos(theta);
    el('line', { x1: ballX, y1: ballY, x2: zblX, y2: zblY,
      stroke: 'rgba(255,255,255,0.90)', 'stroke-width': '2', 'stroke-dasharray': '4 3' });
  }

  // 3. ZBL dot
  el('circle', { cx: zblX, cy: zblY, r: '5', fill: 'white', opacity: '0.95' });

  // 4. Labels
  function label(x, y, text, fontSize) { /* SVG text with dark stroke for contrast */ }

  label(C, zblY - 12, 'ZBL', 8);
  label(C + (-breakSign) * 15, (C + zblY) / 2, `${Math.round(zblAimBase)}"`, 10);
}
```

---

### Result panel — clock mode

The aim cell shows the lateral aim and ZBL reference in text rather than on the SVG.

```js
// When lateralAim > 0:
<div class="result-item">
  <div class="result-value">${lateralAim}" out</div>
  <div class="result-label">Aim ${breakDir}</div>
  <div class="result-zbl-ref">ZBL ${Math.round(zblAimBase)}"</div>
</div>

// When lateralAim === 0:
<div class="result-item">
  <div class="result-value result-value-word">Straight</div>
  <div class="result-zbl-ref">ZBL ${Math.round(zblAimBase)}"</div>
</div>
```

[mobile.css](code://css/mobile.css) — new classes:

```css
.result-zbl-ref {
  font-size: 0.7rem;
  font-weight: 600;
  color: var(--gray-600);
  opacity: 0.75;
  letter-spacing: 0.01em;
  margin-top: 2px;
}

/* Used for word values like "Straight" to prevent wrapping on narrow screens */
.result-value-word {
  font-size: 2rem;
}
```

Also updated `.result-value`:

- `line-height: 1` → `1.15` (prevents descender clipping on letters like `g`)
- Removed `word-break: break-all` (was splitting words mid-character)

---

### Result panel — non-clock mode

- Variance (`+X" / −Y"`) removed from the normal-mode ZBL aim cell
- Uphill/downhill slope row centered (`justify-content: center`) instead of split to opposite ends

---

### Visual summary

| State | Clock face | Result panel |
| --- | --- | --- |
| No distance entered | Empty clock | Empty |
| Distance + slope entered, no clock selected | ZBL dot + measurement line at 12 o'clock | Backswing + ZBL aim |
| Clock selected, straight putt | ZBL dot + measurement line + dashed ball line | Backswing + `Straight` + `ZBL X"` |
| Clock selected, breaking putt | ZBL dot + measurement line + dashed ball line | Backswing + `X" out` + `Aim R→L` + `ZBL X"` |

---

### Files changed

| File | Change |
| --- | --- |
| [`[mobile.css](code://css/mobile.css)`](code://css/mobile.css) | Enlarge `.clock-face` to 280×280; `.clock-hand` height 95px; `.clock-pos` 38px buttons; add `#clock-svg` rule; add `.result-zbl-ref`; add `.result-value-word`; fix `.result-value` line-height; center `.slope-row` |
| [`[mobile.js](code://js/mobile.js)`](code://js/mobile.js) | Update `R`/`C` in `initClockFace()`; append SVG; add `drawClockAnnotations()`; clock aim cell shows `X" out` / `Straight` + ZBL sub-line; non-clock branch passes `zblAimBase` to `drawClockAnnotations` |