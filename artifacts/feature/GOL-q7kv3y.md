---
id: GOL-q7kv3y
type: feature
title: Swipe gestures for onboarding slide navigation
status: Done
---
# Swipe gestures for onboarding slide navigation

## Implementation

Added touch swipe gesture support to the onboarding slides in both the React and vanilla JS implementations.

**Behavior:**

- Swipe **right** → go back to the previous slide
- Swipe **left** → advance to the next slide (same as tapping "Next")
- A minimum 40px horizontal delta is required to register as a swipe, preventing accidental triggers from taps or small movements

**Files changed:**

- [OnboardingModal.tsx](code://src/components/OnboardingModal.tsx) — added `useRef` to track `touchstart` X position, with `handleTouchStart` / `handleTouchEnd` handlers attached to `.onboarding-slides`. Swipe left advances the slide; swipe right goes back (when not on the first slide).
- [mobile.js](code://js/mobile.js) — added equivalent `touchstart` / `touchend` listeners on the overlay element inside `initOnboarding()`, using `{ passive: true }` for scroll performance. Same 40px threshold and direction logic.