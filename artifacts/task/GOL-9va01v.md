---
id: GOL-9va01v
type: task
title: Extrapolate Bryson chart to 1 foot, 2 feet, 3, etc up to 25 feet
---
# Extrapolate Bryson chart to 1 foot, 2 feet, 3, etc up to 25 feet

For the Bryson calibrated chart, I want to see the vectors for all feet up to 25 feet but highlight the original ones (2.5, 5, 7.5, 10, 15, 20, 25).

## Implementation Plan

### Overview

Expand the Bryson table from 7 rows (2.5, 5, 7.5, 10, 15, 20, 25 ft) to 25+ rows covering every foot from 1–25, plus the original half-foot values (2.5, 7.5). Interpolated rows will appear in the table alongside the originals, with the original calibrated rows visually highlighted.

### Current State

- **Data**: `data/putting.json` → `brysonTable.rows` — 7 rows with `steps`, `feet`, `pct1`–`pct6` fields
- **Renderer**: `js/app.js` → `renderBrysonTable()` — maps rows to `<tr>` elements in an HTML table
- **Styling**: `css/styles.css` → `.bryson-table` — centered text, 0.78rem font, bold steps column
- **Cell value formats**: Three formats exist in the data:
- Simple number: `"3.5"` (just break amount in inches)
- Base with negative variance: `"9 −1.5"`
- Base with both variances: `"4 +1.5/−1.5"`

### UX Mock

Open the live HTML mock in a browser to preview the proposed design:

[mock-bryson-expanded.html](code://mock-bryson-expanded.html)

> Run `npx serve` from the project root and navigate to the mock, or open the file directly.

The mock shows all 27 rows (1–25 ft plus 2.5 and 7.5) with the proposed visual treatment:

- **Original rows** → light green background (`--green-pale: #e8f0e4`), bold text, full data with ± variances
- **Interpolated rows** → white background, subdued gray text (`--text-secondary: #555550`), simplified values (no ± shown when variance rounds to 0)
- Steps column shows decimals for interpolated rows (0.4, 0.8, 1.2...) and whole numbers for originals (1, 2, 3...)
- Uses the real app stylesheet so spacing, fonts, scroll behavior, and hover states are accurate

### Changes by File

#### 1. `data/putting.json` — Add interpolated rows

Add rows for every integer foot from 1–25, plus 2.5 and 7.5 (the original half-foot values). Each row gets an `"original": true/false` flag.

- **Original rows** (kept as-is, add `"original": true`): 2.5, 5, 7.5, 10, 15, 20, 25
- **New interpolated rows** (add `"original": false`): 1, 2, 3, 4, 6, 7, 8, 9, 11, 12, 13, 14, 16, 17, 18, 19, 21, 22, 23, 24

**Interpolation method**: Piecewise linear interpolation between the nearest original data points. For each percentage column:

1. Parse the base value (and ± variances if present) from the bracketing original rows
2. Linearly interpolate base value, high variance, and low variance independently
3. Round the base value to the nearest 0.5 inch (per the table's rounding convention)
4. Round variances to the nearest 0.5 inch
5. Format back into the display string (omit ± when variance rounds to 0)

**Steps column**: Compute as `feet / 2.5`, displayed as a decimal rounded to 1 place (e.g., 1 ft → 0.4 steps, 3 ft → 1.2 steps). The original rows remain whole numbers.

**Total rows**: 27 (feet: 1, 2, 2.5, 3, 4, 5, 6, 7, 7.5, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25)

#### 2. `js/app.js` — Update `renderBrysonTable()` to apply highlight class

Update the row-rendering logic to check the `original` flag and apply a CSS class:

```js
const rows = data.rows.map(r => `
  <tr class="${r.original ? 'row-original' : 'row-interpolated'}">
    <td class="cell-steps">${r.steps}</td>
    <td>${r.feet}</td>
    <td>${r.pct1}</td>
    <td>${r.pct2}</td>
    <td>${r.pct3}</td>
    <td>${r.pct4}</td>
    <td>${r.pct5}</td>
    <td>${r.pct6}</td>
  </tr>
`).join('');
```

#### 3. `css/styles.css` — Add highlight styling for original rows

Add styles to visually distinguish the original calibrated rows from the interpolated ones:

```css
/* Original Bryson data rows — highlighted */
.bryson-table tr.row-original td {
  background: var(--green-pale);
  font-weight: 700;
}

/* Interpolated rows — subdued */
.bryson-table tr.row-interpolated td {
  color: var(--text-secondary);
}
```

This gives the original rows a light green background with bold text, while interpolated rows use a lighter text color to visually recede. The table remains fully readable with a clear visual hierarchy.

### Interpolation Calculation Details

Using piecewise linear interpolation with the known anchor points. Example for `pct1` (1% slope):

| Feet | pct1 (inches) | Source |
|------|---------------|--------|
| 2.5  | 1             | Original |
| 5    | 2             | Original |
| 7.5  | 3.5           | Original |
| 10   | 4.5           | Original |
| 15   | 7             | Original |
| 20   | 11            | Original |
| 25   | 14.5          | Original |

For 1 ft (extrapolated below 2.5): Linear extrapolation from 2.5→5 segment gives slope = (2−1)/(5−2.5) = 0.4"/ft, so 1ft ≈ 1 − 0.4×1.5 = 0.5" → rounds to 0.5"

For 3 ft (between 2.5 and 5): interpolation gives 1 + (2−1)×(3−2.5)/(5−2.5) = 1.2" → rounds to 1"

For 12 ft (between 10 and 15): interpolation gives 4.5 + (7−4.5)×(12−10)/(15−10) = 5.5" → rounds to 5.5"

And so on for all columns. The ± variance values are interpolated the same way where they exist; for cells where the bracketing points transition from no-variance to variance, the variance is linearly ramped from 0.

### Edge Cases

- **Below 2.5 ft** (only row 1 and 2): Extrapolate downward from the 2.5→5 segment. Values will be small and simple (no ± variance expected).
- **Variance thresholds**: When interpolated variance rounds to 0, display just the base number (no "+0/−0" clutter).
- **Table width**: The table already scrolls horizontally (`.table-scroll`), so additional rows don't affect width — only height increases, which is fine for a mobile scroll.

### Testing

- Verify the table renders all 27 rows in correct foot order
- Verify original rows (2.5, 5, 7.5, 10, 15, 20, 25) have the highlighted styling
- Verify interpolated values are reasonable (monotonically increasing per column, consistent with surrounding originals)
- Test horizontal scroll still works on mobile-width screens
- Spot-check a few interpolated values against manual calculation