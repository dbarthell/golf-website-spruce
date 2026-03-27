---
id: GOL-fq3f1n
type: memo
title: Version 1.2.0 Release Notes
---
# Version 1.2.0 Release Notes

## Features

### Clockface Slope Tilt ([GOL-eftenh](artifact://GOL-eftenh))

The clock face now tilts in 3D as slope increases, giving a player's-eye view of the green's grade. A CSS `perspective` + `rotateX` transform is applied to the `.clock-face` container proportional to slope (`tiltDeg = slope × 3.5`, so 6% slope → ~21° tilt), with a smooth transition animation as the value changes.

### Lateral Aim Target Ring ([GOL-eftenh](artifact://GOL-eftenh))

A gold diamond marker now appears on the clock SVG at the geometrically correct lateral aim point — the spot perpendicular to the ball-to-hole line at the ZBL lateral aim distance. This is the visual anchor for where the golfer should aim their putter face. The marker is accompanied by a label showing the "out" distance (e.g., "5.2 out") when the aim point is outside the cup, or a named aim label (e.g., "Right Edge") when inside. Only rendered when there is meaningful break (breakAbs ≥ 0.05).

### Swipe Gestures for Onboarding ([GOL-q7kv3y](artifact://GOL-q7kv3y))

Onboarding slides now support touch swipe navigation: swipe left to advance, swipe right to go back. A 40px minimum swipe threshold prevents accidental triggers. Implemented in both the React `OnboardingModal` component and the vanilla `mobile.js` layer.

## Bug Fixes / Infrastructure

- **App Store icon submission fix** — restored all required icon sizes and `CFBundleIconName` after they were inadvertently removed, unblocking App Store submission.
- **Capacitor base path fix** — corrected the Vite base path for Capacitor builds so the iOS app loads assets correctly.
- **Instagram profile SVG** — added an Instagram profile SVG asset for use in marketing/social links.