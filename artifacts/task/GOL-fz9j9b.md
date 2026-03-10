---
id: GOL-fz9j9b
type: task
title: onboarding flow enhancements
status: Done
---
# onboarding flow enhancements

let’s update the onboarding flow; right now there isn’t enough information about 1. what vector putting is and 2. why we calibrate their backswing. the calibration isn’t really connected to the original vector putting research but it actually fits very well with it. i don’t think we should call out 12 inches specifically either in the onboarding flow since that changes with every person.

## Implementation

### Onboarding slides — [OnboardingModal.tsx](code://src/components/OnboardingModal.tsx)

**Slide 1 — "The method" / "Aim above the hole"**
- Grounds ZeroBreak in H.A. Templeton's 1984 book *Vector Putting*, explaining the Zero Break Line concept and the aiming point
- Keeps "ZeroBreak" as the product name; "Vector Putting" is introduced as the research it's built on

**Slide 2 — "Your backswing" / "Trail foot landmarks"**
- Removed the specific "12 inches" measurement (varies per person)
- Frames trail foot landmarks as a distance control solution that pairs with Vector Putting's aim solution
- Explains the why: fixed physical anchor + consistent effort = backswing length controls distance

**Slide 3 — "Make it yours" / "Calibrate once"**
- Uses acceleration profile / constant effort language (from the source research)
- Copy: *"When you apply the same acceleration profile every time — constant effort, consistent cadence — backswing length is what controls distance. Calibrate once and every distance adjusts to your natural pace."*

---

### Calibrate page — [CalibratePage.tsx](code://src/pages/CalibratePage.tsx)

- **Intro copy** moved inside the white tool card with a bottom divider; rewritten as two short paragraphs with explicit instructions including "rolls the ball 1–2 feet past the cup" and a ruler/measuring tape tip (future: swap for ZeroBreak ruler reference)
- **Removed** the "Your Stroke" status card and baseline comparison column — simplified to just "My Backswing" +/− control
- **Removed** the preview table initially; brought it back post-save as "Your Distances" (single backswing column, no baseline) to give users the payoff moment after calibrating
- **First-time users** (from onboarding) stay on the calibrate page after saving and see a "Go to App" button instead of auto-redirecting
- **Default backswing** starts at 10" for uncalibrated users
- **Green speed hint** added below the stimp selector: *"Not sure? 9 is typical for most public courses."* — styled in `--gray-600` for readability
- **Stimp sync**: calibrate page reads and writes `putt-stimp` to localStorage on mount and on change, so stimp stays in sync with the main app page in both directions
- **Calibration math fix**: baseline now uses `findBackswingInches(testDist + 1)` and preview uses `backswingRaw()` to match exactly how CommonDistances computes values on the main page