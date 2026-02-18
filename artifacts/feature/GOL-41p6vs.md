---
id: GOL-41p6vs
type: feature
title: Add numbers for lag putting
status: In Progress
---
# Add numbers for lag putting

I’m working on a system for lag putts like Bryson where I base the length of the putt by how far I take my putter back relative to my right foot which is 12 inches to the right of the ball. I found that 15 inches (putter face to right edge of foot) is roughly a 20 foot putt on hibiscus golf putting green in Naples FL so maybe 9 stimp. How would you extrapolate that to some other length putts and backswing length.

distance is roughly square of backswing length i think.

You are absolutely correct. While many golfers use a linear "inches-to-feet" mental model, the 

**physics of a pendulum** actually dictates that the energy (and thus the distance) is proportional to the **square** of the backstroke length.

In physics terms: . Since the height  the putter rises is related to the square of the arc length, doubling your backswing doesn't just double your distance—it can quadruple it.

If we apply this **Inverse Square Law** to your confirmed data points at **Hibiscus (Stimp 9)**, the math gets very interesting.

**The Quadratic Calibration (Stimp 9)**

Using your baseline where **15 inches = 20 feet**, the formula for your specific "Smooth Pendulum" tempo is roughly:

Backstroke (Inches)MarkerRoll Distance (Feet)

**4.5"**

One Putter Head

**\~2 feet**

**8.0"**

Inside Heel

**\~5.5 feet**

**9.0"**

Inside Trail Foot

**\~7 feet**

**12.0"**

**Middle Trail Foot**

**\~13 feet**

**15.0"**

**Outside Trail Foot**

**20 feet (Your Baseline)**

**18.0"**

3" Past Outside

**\~29 feet**

**21.0"**

6" Past Outside

**\~39 feet**

**Why this "Square Rule" explains your feel:**

1. **The "Slow" Short Putts:** If you use the square rule, your **8-9 inch** strokes (inside foot) will feel much "shorter" than a linear model would suggest. This matches why many players struggle with leaving 8-footers short—they don't realize how much more energy is required compared to a tap-in.
2. **The "Explosion" on Lags:** Notice how jumping from **15 inches** (outside foot) to **21 inches** (6" past) doesn't just add 6 feet of roll—it potentially adds **nearly 20 feet**. This is why long lag putts at Hibiscus can get away from you so quickly if your backswing gets too long.
3. **Tempo is the Constant:** For this squared math to work, your **tempo must stay the same**. If you take a 21-inch backswing but "slow down" your hands to compensate, you’re fighting the physics of the pendulum.

**The Takeaway for your Round**

If you believe in the **square of the backswing**, your "Middle of Foot" (12") is your **13-foot marker**, not 16. This means for anything over 15 feet, you **must** be outside your trail foot.

**✅ Answer**
Based on the square of the backswing:

- Your **4.5-inch** stroke (putter head) rolls about **2 feet**.
- Your **12-inch** stroke (middle foot) rolls about **13 feet**.
- Going **9 inches outside your trail foot** (24" total) would theoretically send the ball **over 50 feet**.

Does this "exponential" increase at the longer distances match what you see when you're hitting those long lags across the green?

---

## Implementation Plan

### Overview

Add a **Lag Putting** card to the existing Putting section. It will display a reference table showing 7 backswing-length landmarks (in inches, relative to the trail foot) and their corresponding roll distances, derived from the quadratic pendulum formula calibrated at Hibiscus Golf (Stimp 9).

**Formula:** `distance (ft) = (20 / 225) × backswing²`
*(calibrated from 15" = 20 ft baseline)*

---

### Files to Change

| File | Change |
|---|---|
| `data/putting.json` | Add `lagPuttingTable` key with 7 data rows |
| `js/app.js` | Add `renderLagPuttingTable()` function; call it from `renderPutting()` |
| `css/styles.css` | Add styles for `.lag-table` |

---

### 1. `data/putting.json` — Add `lagPuttingTable`

Add a new top-level key alongside `brysonTable`, `zblChart`, and `gravityVector`:

```json
"lagPuttingTable": {
  "title": "Lag Putting — Backswing Length",
  "description": "Stimp 9 (Hibiscus) — Square-law calibration. 15\" outside trail foot = 20 ft baseline.",
  "columns": ["Backswing", "Landmark", "Roll Distance"],
  "rows": [
    { "inches": "4.5\"", "landmark": "One Putter Head",    "feet": "~2 ft",   "original": false },
    { "inches": "8.0\"", "landmark": "Inside Heel",        "feet": "~5.5 ft", "original": false },
    { "inches": "9.0\"", "landmark": "Inside Trail Foot",  "feet": "~7 ft",   "original": false },
    { "inches": "12.0\"","landmark": "Middle Trail Foot",  "feet": "~13 ft",  "original": false },
    { "inches": "15.0\"","landmark": "Outside Trail Foot", "feet": "20 ft",   "original": true  },
    { "inches": "18.0\"","landmark": "3\" Past Outside",   "feet": "~29 ft",  "original": false },
    { "inches": "21.0\"","landmark": "6\" Past Outside",   "feet": "~39 ft",  "original": false }
  ]
}
```

The `original: true` row (15" baseline) will be visually highlighted using the existing `row-original` CSS pattern already used by the Bryson table.

---

### 2. `js/app.js` — New render function + wire-up

**Add** `renderLagPuttingTable(data)` following the same pattern as `renderZBLChart`:

```js
function renderLagPuttingTable(data) {
  const rows = data.rows.map(r => `
    <tr class="${r.original ? 'row-original' : ''}">
      <td class="cell-backswing">${r.inches}</td>
      <td>${r.landmark}</td>
      <td class="cell-distance">${r.feet}</td>
    </tr>
  `).join('');

  return `
    <div class="putting-card">
      <div class="putting-card-header">
        <h3>${data.title}</h3>
        <p>${data.description}</p>
      </div>
      <div class="table-scroll">
        <table class="putting-table lag-table">
          <thead>
            <tr>
              ${data.columns.map(c => `<th>${c}</th>`).join('')}
            </tr>
          </thead>
          <tbody>${rows}</tbody>
        </table>
      </div>
    </div>
  `;
}
```

**Update** `renderPutting()` to render the lag table **first** (it's the new primary lag reference):

```js
function renderPutting(puttingData) {
  const container = document.getElementById('putting-grid');
  let html = '';

  if (puttingData.lagPuttingTable) html += renderLagPuttingTable(puttingData.lagPuttingTable);
  if (puttingData.brysonTable)     html += renderBrysonTable(puttingData.brysonTable);
  if (puttingData.zblChart)        html += renderZBLChart(puttingData.zblChart);
  if (puttingData.gravityVector)   html += renderGravityNote(puttingData.gravityVector);

  container.innerHTML = html;
}
```

---

### 3. `css/styles.css` — Lag table styles

Add after the existing ZBL table styles (around line 416):

```css
/* Lag Putting Table */
.lag-table .cell-backswing {
  font-weight: 700;
  color: var(--green-dark);
  font-variant-numeric: tabular-nums;
  white-space: nowrap;
}

.lag-table .cell-distance {
  font-weight: 700;
  color: var(--green-dark);
  text-align: right;
}

.lag-table tr.row-original td {
  background: var(--green-pale);
  font-weight: 700;
}

.lag-table tr.row-original .cell-backswing {
  font-weight: 800;
}
```

---

### Visual Result

The new card will appear at the top of the Putting section:

```
┌─────────────────────────────────────────┐
│  Lag Putting — Backswing Length         │  ← dark green header
│  Stimp 9 (Hibiscus) — Square-law…      │
├──────────────┬──────────────────┬───────┤
│  Backswing   │  Landmark        │  Roll │
├──────────────┼──────────────────┼───────┤
│  4.5"        │  One Putter Head │  ~2ft │
│  8.0"        │  Inside Heel     │ ~5.5ft│
│  9.0"        │  Inside Trail Ft │  ~7ft │
│  12.0"       │  Middle Trail Ft │ ~13ft │
│ ★15.0"       │  Outside Trail Ft│  20ft │  ← green highlight (baseline)
│  18.0"       │  3" Past Outside │ ~29ft │
│  21.0"       │  6" Past Outside │ ~39ft │
└──────────────┴──────────────────┴───────┘
```

The baseline row (15" / 20 ft) uses the existing `row-original` green-pale highlight, consistent with how calibrated rows are styled in the Bryson table. No new HTML structure or nav changes are needed — the card slots cleanly into the existing putting-grid layout.