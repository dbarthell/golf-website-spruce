---
id: GOL-zkc5fj
type: task
title: Remove the about squiggles (~) from the lag putting table
status: done
---
# Remove the about squiggles (~) from the lag putting table

## Work Done

Removed all `~` (tilde/squiggle) characters from the `lagPuttingTable` rows in `data/putting.json`. All 11 rows had their `stimp9`, `stimp10`, and `stimp11` values cleaned up — e.g. `"~22 ft"` → `"22 ft"`.
