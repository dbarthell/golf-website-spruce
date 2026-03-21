---
id: GOL-s0r8dt
type: feature
title: Backstroke label + golf-green clock face
status: Done
---
# Backstroke label + golf-green clock face

1. Replace "Backswing" with "Backstroke" everywhere in the UI
2. Redesign the clock position area to look like a high-end, manicured golf green

## Implementation

### 1. "Backswing" → "Backstroke" ([LookupResult.tsx](code://src/components/LookupResult.tsx))

Fixed the last remaining user-visible "Backswing" label — the clock-mode result panel. All other panels (straight-mode result, Backstroke Reference table, Common Distances, Calibrate page) were already correct.

### 2. Golf-green clock face

[global.css](code://src/styles/global.css)

- **Mower stripes** — `repeating-linear-gradient` at 15° alternating `var(--green-dark)` / `#0f5038` in 14px bands, matching the app's existing palette.
- **Sunlight highlight** — subtle radial glow at centre using the logo's gold accent (`rgba(201,168,106,0.10)`) fading to transparent.
- **SVG grain fix** — `feTurbulence` fractalNoise (4 octaves) with `feColorMatrix` correctly mapping the noise B-channel to alpha (0–28% opacity) for genuine per-pixel grass-texture variation.
- **Gold bezel** — border changed to `rgba(201,168,106,0.5)` with a matching inner glow, tying the rim to the logo's brass/gold accent.
- **Tapered clock hand** — gradient from dim at the pivot to bright in the shaft with a soft white glow; reads as a real instrument hand.
- **Frosted-glass position buttons** — `backdrop-filter: blur(4px)` lets the stripe pattern show through instead of a flat opaque circle.
- **Gold active state** — selected position lights up in `var(--green-accent)` gold with a halo, matching the logo rather than blowing out to white.
- **Cardinal tick marks** — subtle gold `<line>` elements at 12/3/6/9 in the outer SVG ring, like a quality watch bezel.
- **Cup ring** — faint gold halo around the centre pivot suggesting the actual hole on the green.
- **Surrounding band** — `.clock-section` uses `var(--green-dark)` (`#013c36`), matching the logo background exactly.

### 3. Follow-up polish ([CalibratePage.tsx](code://src/pages/CalibratePage.tsx), [global.css](code://src/styles/global.css))

- **"Test distance" label** — renamed the distance chip selector label on the Calibrate page from "Distance" to "Test distance" for clarity.
- **:30 button parity** — removed the explicit `width: 34px`, `height: 34px`, and `opacity: 0.85` overrides from `.clock-pos-half` so 1:30, 4:30, 7:30, and 10:30 buttons match the size and opacity of all other clock position buttons.