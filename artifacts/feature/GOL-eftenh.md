---
id: GOL-eftenh
type: feature
title: Clockface slope tilt and lateral aim target ring
status: Done
---
# Clockface slope tilt and lateral aim target ring

I want to make two enhancements to the app:

1. i want the clockface to dynamically change when the slope changes to indicate the clockface is more (or less) sloped as if it’s a real green
2. i want to add a target, visual, indicator, etc. along the dotted line that maps to the big lateral aim “out” number since that’s where the user conventionally aims their putt (i.e. in relation to the hole), which will always be a right angle from the straight line to the dotted line

---

## Implementation Plan

### How the Current Code Works

**Clockface rendering** ([ClockFace.tsx](code://src/components/ClockFace.tsx)): A 280×280px `.clock-face` div rendered with two SVG layers and absolutely-positioned HTML buttons. The first SVG contains the grass texture, tick marks, and pivot; the second (`ClockSVG`) renders annotations (fall-line dashes, ZBL measurement line, dotted break line, ZBL dot, labels).

**Slope** flows in as a numeric prop and currently only drives the `↓ X%` dashed fall-line drawn straight down the SVG center (12→6 axis). The clock face itself stays flat regardless of slope.

**Lateral aim calculation** ([LookupPage.tsx](code://src/pages/LookupPage.tsx), line 70):
```
lateralAim = (zblAimBase + varianceAdj) × breakAbs
edgeAim    = lateralAim − 2.125   ← “big out number” shown in results
```
`zblAimBase` is the Bryson-table total aim; `breakAbs = |sin(clockTheta)|` is how much of the putt is pure lateral vs. uphill/downhill. `lateralAim` and `edgeAim` are already in `ClockAnnotations` and passed to `ClockFace`, but nothing uses them visually on the clock itself yet.

**The dotted line** runs from `(ballX, ballY)` (the selected clock-hour button position) to `(SVG_C, SVG_C − zblPx)` (the ZBL dot on the 12-o’clock axis). The ZBL dot is always on the vertical center axis — a deliberate simplification. `breakFactor` (signed, indicating R→L vs. L→R) is computed in `LookupPage` but **not** currently included in `ClockAnnotations`.

---

### Enhancement 1: Tilt the Clockface with Slope

**Approach — CSS 3D perspective transform**

Apply a `rotateX` tilt to the `.clock-face` container using `perspective()`. The green always tilts toward the viewer along the fall line (12 is highest/farthest, 6 is lowest/nearest), making it look like you’re looking at a real sloped green from a player’s-eye view.

**Tilt formula:**
```
tiltDeg = slope × 3.5     // 0% → 0°, 6% → 21°
```
*(The exact multiplier is a UX tuning decision — see open questions.)*

**Files to change:**

1. **[ClockFace.tsx](code://src/components/ClockFace.tsx)**
   - Compute `tiltDeg` from the `slope` prop
   - Wrap the `.clock-face` div in a perspective container, or apply the transform directly on `.clock-face`:
     ```tsx
     style={{ transform: `perspective(500px) rotateX(${tiltDeg}deg)` }}
     ```
   - Keep `transform-origin: center center` so the tilt pivots around the hole.

2. **[global.css](code://src/styles/global.css)**
   - Add `transition: transform 0.35s ease` to `.clock-face` for smooth animation as slope changes.
   - Add `transform-style: preserve-3d` if child elements need to participate in the 3D space (likely not needed since SVG/buttons stay flat).

**No data or calculation changes required** — `slope` is already a prop on `ClockFace`.

**Open questions for human review:**
- [ ] Multiplier feel: does `slope × 3.5` (6% = 21°) look right? Too subtle / too dramatic?
- [ ] Should 0% slope show completely flat (no tilt at all)? Or a very slight default tilt for aesthetics?
- [ ] Tilt direction confirmed as “top away, bottom near” (like standing at the 6 o’clock side and looking uphill toward the hole)?

---

### Enhancement 2: Lateral Aim Target Indicator

**Goal:** Add a distinct visual marker on the clockface SVG that sits at the geometrically correct “aim point” — the location `lateralAim` inches from the hole center, perpendicular to the ball-to-hole line (the high side). This is the spot the user aims their putter face toward.

**Geometry:**

The aim target is the point reached by starting at the hole center `(SVG_C, SVG_C)` and moving perpendicular to the ball-to-hole direction by `scaleIn(lateralAim)` SVG pixels, toward the high side. Using the visual theta `θ` from `HOUR_POSITIONS`:

```
Ball-to-hole unit direction:  (-sin θ,  cos θ)   [in SVG space, y-down]
High-side perpendicular:       (-cos θ, -sin θ)  × breakSign
                               where breakSign = sign(breakFactor)

aimIndicatorX = SVG_C + (-cos θ) × breakSign × scaleIn(lateralAim)
aimIndicatorY = SVG_C + (-sin θ) × breakSign × scaleIn(lateralAim)
```

For cardinal positions (3 o’clock, 9 o’clock) where `breakAbs = 1`, this lands exactly on the existing ZBL dot. For diagonal positions, it sits between the hole and the ZBL dot, slightly off the dotted line (the dotted line is a visual approximation anyway).

**Verification (3 o’clock, θ = π/2, breakFactor = +1):**
- `aimX = SVG_C + (−cos π/2) × 1 × px = SVG_C + 0 = SVG_C` ✓
- `aimY = SVG_C + (−sin π/2) × 1 × px = SVG_C − px` (above hole, toward 12) ✓
- Perpendicular to ball-to-hole direction `(−1, 0)`: dot product `= 0` ✓

**Data change required — add `breakFactor` to `ClockAnnotations`:**

Currently `ClockAnnotations` has `breakAbs` (unsigned) but not the signed `breakFactor`. We need the sign to know which side of the hole the indicator falls on.

**Files to change:**

1. **[types.ts](code://src/lib/types.ts)**
   - Add `breakFactor: number` to `ClockAnnotations`:
     ```ts
     export interface ClockAnnotations {
       zblAimBase: number;
       lateralAim: number;
       breakAbs: number;
       breakFactor: number;   // ← new, signed; 0 if no break
       clockKey: string | null;
     }
     ```

2. **[LookupPage.tsx](code://src/pages/LookupPage.tsx)**
   - In `computeResult`, include `breakFactor` in the `annotations` object (line 76):
     ```ts
     const annotations: ClockAnnotations = {
       zblAimBase, lateralAim, breakAbs,
       breakFactor,          // ← add this
       clockKey: clock,
     };
     ```
   - Also update the straight-mode annotations (line 114) with `breakFactor: 0`.
   - Update the `annotations` fallback in the page component (line 167) to include `breakFactor: 0`.

3. **[ClockFace.tsx](code://src/components/ClockFace.tsx)** — `ClockSVG` function
   - Pull `lateralAim` and `breakFactor` from `annotations`.
   - Compute the indicator position using the formula above.
   - Render a distinct marker — proposed: a **gold/amber diamond** (`◆`) or rotated `<rect>` to distinguish it visually from the white ZBL dot.
   - Add a label showing the `edgeAim` value when outside the cup (i.e., when `lateralAim > 2.125`), or the named aim point text when inside (`Right Edge`, etc.) — mirroring what `LookupResultPanel` already shows in the result card.
   - Only render when `hasBreak && breakAbs >= 0.05`.

   Sketch of the new SVG block inside `ClockSVG`:
   ```tsx
   {/* Lateral aim indicator */}
   {hasBreak && lateralAim > 0 && (
     <>
       <rect
         x={aimX - 5} y={aimY - 5}
         width={10} height={10}
         transform={`rotate(45, ${aimX}, ${aimY})`}
         fill=”#c9a86a”    // gold, matches the clock bezel accent color
         opacity=”0.95”
       />
       <text {...textProps} x={aimX + labelOffsetX} y={aimY} fontSize=”9”>
         {edgeAim > 0 ? `${fmtInches(edgeAim)} out` : namedAimLabel}
       </text>
     </>
   )}
   ```

   *Note: `edgeAim` and a named aim label will need to be derived from `lateralAim` and `breakFactor` inside `ClockSVG`, or passed via `ClockAnnotations`. Simplest path: compute `edgeAim = lateralAim − 2.125` locally in the SVG component since all inputs are already available.*

**Open questions for human review:**
- [ ] Visual style: gold diamond confirmed, or prefer a different shape/color (circle, crosshair, flag pin icon)?
- [ ] Label content: show `edgeAim` in inches (“5.2 out”) or the named aim point (“Right Edge”) or both?
- [ ] When `edgeAim ≤ 0` (aim point is inside the cup), should the indicator still render? The named aim point is inside the cup — it might be confusing to show a marker at sub-cup distances on the clock.
- [ ] Should the dotted line be re-routed to pass through the aim indicator (ball → indicator → ZBL dot), making it literally “along the dotted line”? Or is a free-floating indicator acceptable?

---

### Summary of Files Changed

| File | Change |
|---|---|
| [ClockFace.tsx](code://src/components/ClockFace.tsx) | Add CSS 3D tilt from `slope`; add lateral aim indicator marker + label in `ClockSVG` |
| [global.css](code://src/styles/global.css) | Add `transition` + `transform-style` to `.clock-face` |
| [types.ts](code://src/lib/types.ts) | Add `breakFactor: number` to `ClockAnnotations` |
| [LookupPage.tsx](code://src/pages/LookupPage.tsx) | Include `breakFactor` in all `ClockAnnotations` construction sites (3 places) |

### Sequence of Implementation Steps

1. **Add `breakFactor` to `ClockAnnotations`** (`types.ts`) and fix all three callsites in `LookupPage.tsx` — TypeScript will catch any missed spots at compile time.
2. **Implement the clockface tilt** (`ClockFace.tsx` + `global.css`) — isolated, no data changes, easy to tune visually.
3. **Implement the lateral aim indicator** in `ClockSVG` — verify geometry at 3, 9, 1:30, and 10:30 positions.
4. **(Optional)** Reroute dotted line through the indicator if desired after visual review.