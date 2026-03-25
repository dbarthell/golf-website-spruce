---
id: GOL-2aa1va
type: feature
title: UI enhancements
status: Done
---
# UI enhancements

Let’s try making the whole top of the main green instead of green header, white selectors, then green clock

## Implementation

Unified the entire top section of the Lookup page into a single continuous dark-green band (matching `--green-dark: #013c36`). Previously the layout was: green header → white inputs panel → green clock. Now it’s: green header → green inputs → green clock — one seamless block.

### Changes

**[global.css](code://src/styles/global.css)**
- Changed `.lookup-body` background from `var(--white)` to `var(--green-dark)`
- Removed the white-background overrides for `.lookup-body .distance-input`, `.slope-input`, `.input-btn` — the inputs now use their default dark-hero styles (white text, semi-transparent borders)
- Updated `.lookup-body .lookup-label` color to `rgba(255, 255, 255, 0.7)` to match the clock section label style

**[LookupPage.tsx](code://src/pages/LookupPage.tsx)**
- Removed `variant="light"` from `<StimpToggle>` so it uses the default `"dark"` variant (gold indicator, white labels on green background)