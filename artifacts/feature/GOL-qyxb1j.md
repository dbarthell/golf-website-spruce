---
id: GOL-qyxb1j
type: feature
title: Double putting numbers
status: Done
---
# Double putting numbers

Let’s extrapolate both the lag putting and calibrated bryson table to longer distances. Let’s do 90 feet for lag putting and 50 feet for bryson table. we can go in 5 foot increments this time so 30, 35, 40, 45, 50 for the bryson table.

## Work Done

Extended both putting tables in `data/putting.json` using linear extrapolation from existing calibration points.

**Lag Putting Table** — extended from ~39 ft to ~91 ft by continuing the 3" backswing increment pattern:
| Backswing | Landmark | Roll Distance |
|-----------|----------|---------------|
| 24.0" | 9" Past Outside | ~51 ft |
| 27.0" | 12" Past Outside | ~65 ft |
| 30.0" | 15" Past Outside | ~80 ft |
| 32.0" | 17" Past Outside | ~91 ft |

Formula: `distance = 20 × (backswing / 15)²` (square-law, Stimp 9 baseline)

**Bryson Table** — added 5 new rows at 5-ft increments (30–50 ft) using extrapolation from the 20 ft and 25 ft anchor points. Also updated `scripts/interpolate-bryson.js` to support beyond-last-anchor extrapolation and include 30–50 ft in target feet.