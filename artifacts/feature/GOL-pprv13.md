---
id: GOL-pprv13
type: feature
title: Main screen header hidden above viewport on first launch
status: Done
---
# Main screen header hidden above viewport on first launch

On first launch after a fresh install, the onboarding flow navigates to the Calibrate page. After scrolling down on Calibrate and tapping "Go to App", React Router changes the route but doesn't reset the scroll position — so the main screen loads with the window already scrolled past the header, requiring the user to scroll up to see it.

## Fix

Added a `useEffect` in [LookupPage.tsx](code://src/pages/LookupPage.tsx) that calls `window.scrollTo(0, 0)` on mount. This ensures the main screen always starts at the top whenever the user navigates to it, from any page.