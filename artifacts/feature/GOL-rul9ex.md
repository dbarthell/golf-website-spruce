---
id: GOL-rul9ex
type: feature
title: Ruler Export — ZeroBreak Calibration Kit
status: Backlog
priority: High
size: L
tasks: []
---
# Ruler Export — ZeroBreak Calibration Kit

A "Print / Export Your Ruler" screen that generates a personalized ruler specification from the user's `distanceFactor` and preferred Stimp. Bridges the digital calibration to a planned physical laser-engraved ruler product.

## Background

The business model: calibrate on the app → app generates personal stroke zones → buy a custom ruler with those zones pre-marked. Two outputs are needed:

1. **Printable SVG ruler** — 1:1 scale (96 DPI), tiled across 8.5×11 landscape pages, usable immediately as a "Starter Ruler"
2. **Order Summary Card** — shareable spec sheet showing the Zone Map (e.g., *10 ft = 4.5"*) for attaching to a custom-engraver order

## Zone Map Logic

For each common distance (5, 6, 10, 15, 20, 25, 30, 40, 50 ft) at the user's saved Stimp, compute `backswingDisplay(dist, stimp, lagRows, distanceFactor)`. These are the inch marks that get laser-etched. All data already exists in the app — the export just formats it for print/manufacturing.

## Acceptance Criteria

- A new `/ruler` route renders `RulerPage`
- `CalibratePage` shows a "Get Your Ruler →" navigation link
- `RulerPage` displays the Zone Map table (distance → personal backswing inch mark)
- Trail-foot landmark zones (inside foot ≈ 9", middle ≈ 12", outside foot ≈ 18") are shown as named labels, not raw inch values
- "Download Ruler (SVG)" exports an SVG at exactly 1 inch = 96px so it prints 1:1 from a browser print dialog
- SVG is DXF-compatible (vector paths only — no raster elements) for laser engraving hand-off
- A QR code (lightweight client-side library, e.g. `qrcode`) is embedded in the SVG, deep-linking to `/calibrate`
- "Order a Custom Engraved Ruler" button links to an order form (URL TBD — stub with a placeholder for now)
- Free tier: printable paper SVG (no gate)
- Premium upsell: "Order a Custom Engraved Ruler" CTA is visible and distinct

## Key Files

- New: [RulerPage.tsx](code://src/pages/RulerPage.tsx)
- New: [rulerExport.ts](code://src/lib/rulerExport.ts)
- Edit: [App.tsx](code://src/App.tsx) — add `/ruler` route
- Edit: [CalibratePage.tsx](code://src/pages/CalibratePage.tsx) — "Get Your Ruler →" link
- Dependency: add `qrcode` (or equivalent lightweight QR library)
