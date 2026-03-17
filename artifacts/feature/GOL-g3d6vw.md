---
id: GOL-g3d6vw
type: feature
title: More enhancements
status: Done
---
# More enhancements

## Copy enhancements

1. **Slide 1** — Rewrote with clock face metaphor and "the straight putt" callout:
   > "H.A. Templeton's 1984 book Vector Putting proved that every breaking putt has a Zero Break Line — a straight downhill path through the hole. Imagine a clock face where 6 o'clock is straight downhill; no matter your ball's position, you always aim at a point on that 12 o'clock line — the straight putt. ZeroBreak finds that point for you."
2. **Slide 2** — Rewritten with backstroke, tempo, and traditional vs. scientific framing. Tag updated to "Your backstroke":
   > "Aim is only half the equation. Traditional putting relies on instinct and athleticism to control speed — but instinct alone can't account for every variation in green speed, slope, and (yes) nerves, not without the reps of a tour professional. ZeroBreak takes a scientific approach: it calibrates the length of your backstroke to the length of the putt, tuned to your unique tempo. Build a feel for a few key distances, and you'll find you can step up to any putt with confidence. We've found it's the most reliable way to nail the right speed, every single time."
3. **Slide 3** — New opening sentence, cleaned up em dashes, updated "backswing" to "backstroke":
   > "No two putting strokes are alike. Some give it a good pop, others are slow and deliberate. Calibrate your stroke to a single distance (10 ft is a good starting point) and the app uses your personal acceleration profile to calculate every other distance automatically."
4. **Calibrate screen** — `backswing → backstroke` in three places: intro copy, "My Backstroke" label, preview table header.
5. **Main app** — `backswing → backstroke` in five places: LookupResult labels (straight + clock modes), Backstroke Reference section title and column header, Common Distances subtitle.

## Common distances table

1. Updated range from 10 ft to 100 ft in 5 ft increments.
2. Moved to below the Backstroke Reference table.

## Backstroke Reference table

1. Added assumption note: "Assumes 12″ between ball and trail foot."

## Slope fall-line indicator on clock face

When slope > 0, a subtle dashed fall-line appears on the clock face running from 12 o'clock (uphill) to 6 o'clock (downhill), with a small `↓ {slope}%` label below center-right.

## Walk-off tip

Added a dismissible tip card below the distance/slope inputs:

> "Walk off your putts — 1 step ≈ 3 ft. Practice your stride with a ZeroBreak ruler or tape measure to dial it in."

Dismissed state persisted in localStorage. Won't reappear once dismissed.