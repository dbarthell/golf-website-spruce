---
id: GOL-rmvknv
type: task
title: Change steps to 1 step for every 3 feet
---
# Change steps to 1 step for every 3 feet

## Summary

The Bryson putting table currently uses a **1 step = 2.5 feet** conversion ratio. This task changes it to **1 step = 3 feet**.

## Current Behavior

- Conversion formula in `scripts/interpolate-bryson.js` (line 159): `steps = feet / 2.5`
- Original anchor rows in `data/putting.json` have step values based on the 2.5 ratio (e.g. 1 step = 2.5 ft, 2 steps = 5 ft, etc.)
- The interpolation script reads the original rows, then generates interpolated rows for every foot 1–25, computing step values with the 2.5 divisor

## Desired Behavior

- **1 step = 3 feet** (e.g. 3 feet → 1.0 steps, 6 feet → 2.0 steps, 15 feet → 5.0 steps)
- All step values in the table should be recalculated using the new ratio
- The percentage (break) values remain unchanged — they are keyed to feet, not steps

## Implementation Plan

### Step 1: Update the interpolation script conversion factor

**File:** `scripts/interpolate-bryson.js` (line 159)

Change:

```javascript
const steps = Math.round((feet / 2.5) * 10) / 10; // 1 decimal place
```

To:

```javascript
const steps = Math.round((feet / 3) * 10) / 10; // 1 decimal place
```

### Step 2: Update original anchor rows in `putting.json`

**File:** `data/putting.json`

The original (calibrated) rows have hardcoded `steps` values that the interpolation script copies as-is. These need to be recalculated with `feet / 3`:

| Feet | Old Steps (÷ 2.5) | New Steps (÷ 3) |
|------|-------------------|-----------------|
| 2.5  | 1.0               | 0.8             |
| 5    | 2.0               | 1.7             |
| 7.5  | 3.0               | 2.5             |
| 10   | 4.0               | 3.3             |
| 15   | 6.0               | 5.0             |
| 20   | 8.0               | 6.7             |
| 25   | 10.0              | 8.3             |

### Step 3: Re-run the interpolation script and update data

Run the interpolation script to regenerate the full table with the new step values:

```bash
node scripts/interpolate-bryson.js
```

Then replace the `brysonTable.rows` array in `data/putting.json` with the script output.

### Expected result for all rows after regeneration

| Steps (new) | Feet | Original? |
|-------------|------|-----------|
| 0.3         | 1    | false     |
| 0.7         | 2    | false     |
| 0.8         | 2.5  | true      |
| 1.0         | 3    | false     |
| 1.3         | 4    | false     |
| 1.7         | 5    | true      |
| 2.0         | 6    | false     |
| 2.3         | 7    | false     |
| 2.5         | 7.5  | true      |
| 2.7         | 8    | false     |
| 3.0         | 9    | false     |
| 3.3         | 10   | true      |
| 3.7         | 11   | false     |
| 4.0         | 12   | false     |
| 4.3         | 13   | false     |
| 4.7         | 14   | false     |
| 5.0         | 15   | true      |
| 5.3         | 16   | false     |
| 5.7         | 17   | false     |
| 6.0         | 18   | false     |
| 6.3         | 19   | false     |
| 6.7         | 20   | true      |
| 7.0         | 21   | false     |
| 7.3         | 22   | false     |
| 7.7         | 23   | false     |
| 8.0         | 24   | false     |
| 8.3         | 25   | true      |

### Files Changed

| File | Change |
|------|--------|
| `scripts/interpolate-bryson.js` | Update conversion divisor from `2.5` → `3` |
| `data/putting.json` | Update all step values (original anchors + re-run script for interpolated rows) |

### Notes

- **No changes to&#32;`js/app.js`** — the rendering logic just displays whatever `steps` value is in the data.
- **No changes to percentage/break values** — those are tied to feet, not steps.
- **No changes to CSS** — styling is unaffected.
- The putting data the user sees on the website will still show every foot from 1–25 plus the half-foot anchors (2.5, 7.5). Only the "Steps" column values change.