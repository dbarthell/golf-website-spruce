---
id: GOL-g67n8z
type: feature
title: Golf Intelligence API
status: In Progress
---
# Golf Intelligence API

## Feature Summary

Integrate the Golf Intelligence (GI) API into the ZeroBreak putting module to surface **green slope images** and use them to **automatically infer distance, slope %, and clock position** from a two-tap interface. The golfer taps the hole and their ball on the slope map; the app calculates distance from pixel geometry and infers slope/break direction from the GI color scale.

---

## Implementation Plan

### What Needs to Change and Why

The original plan was just to display a slope image as a visual aid. During implementation, we went further: by reading pixel colors from the GI slope image we can infer the actual slope %, the uphill/downhill direction, and the break direction — eliminating three manual input steps for the golfer. This is the core value of the integration.

### Architecture Overview

```
GreenSlopePanel (UI) ──┐
                       ├── useCourseSelector (state hook)
                       │       ├── golfIntelligence.ts  (API client, auth, caching)
                       │       └── localStorage          (course/hole persistence)
                       └── slopeColor.ts               (pixel color → slope+clock)
```

### Files Created

| File | Purpose |
| --- | --- |
| [golfIntelligence.ts](code://src/lib/golfIntelligence.ts) | GI API client — OAuth token exchange, course search, scorecard (hole IDs), slope/elevation image fetch with two-layer cache (memory + localStorage) |
| [slopeColor.ts](code://src/lib/slopeColor.ts) | Canvas pixel reader — classifies GI color scale to slope %, infers clock position from uphill/downhill and break direction gradients |
| [useCourseSelector.ts](code://src/hooks/useCourseSelector.ts) | React state hook — debounced search, course/hole persistence, explicit image load trigger |
| [GreenSlopePanel.tsx](code://src/components/GreenSlopePanel.tsx) | Collapsible UI panel — course search → hole picker → slope/elevation toggle → two-tap inference → "Use in app" |

### Files Modified

| File | Change |
| --- | --- |
| [LookupPage.tsx](code://src/pages/LookupPage.tsx) | Added `<GreenSlopePanel>` stacked between inputs and WalkOffTip/clock face |
| [vite.config.ts](code://vite.config.ts) | Added `/gi-api` proxy (GI API, avoids CORS) and `/cf-img` proxy (CloudFront CDN, enables canvas `getImageData`) |
| [global.css](code://src/styles/global.css) | Added all GreenSlopePanel styles: toggle header, search, hole chips, mode toggle, image overlay, result panel, clock chips |
| `.env.local` | `VITE_GI_API_TOKEN` and `VITE_GI_CLIENT_ID` (gitignored) |

### Key Technical Decisions

**Auth flow:** GI uses a two-step OAuth-style flow — POST the static API token to `/auth/authenticateToken` to get a short-lived Bearer access token. The client caches the access token + refresh token in memory with a 1-hour TTL. On expiry it tries the refresh token first, falls back to full re-auth.

**Credit conservation:** Slope/elevation images cost 1 credit each. The app only fetches on explicit user tap ("Load Green Map"). Fetched images are cached in two layers: in-memory `Map` (survives re-renders, cleared on page reload) and `localStorage` keyed by `endpoint:holeId` (survives page reloads, \~1-year TTL from CloudFront URL expiry).

**Canvas CORS:** CloudFront blocks `canvas.getImageData()` cross-origin. In the browser we route the image through a Vite `/cf-img` proxy (same-origin). In the Capacitor iOS native build, CORS isn't a restriction so we use the direct URL.

**Clock inference math:**

- `uphillValue = avgSteepness(hole end) − avgSteepness(ball end)` — positive = uphill putt (toward steeper/higher)
- `breakValue = avgSteepness(right side) − avgSteepness(left side)` — positive = right-high = ball breaks left (3 o'clock)
- `theta = atan2(breakValue, −uphillValue)` → mapped to nearest CLOCK\_ENTRIES key

**Tap vs scroll conflict:** `onTouchStart` records position; `onTouchEnd` only registers a tap if movement < 12px. This prevents accidental taps while the user scrolls past the image.

### Remaining Work / Open Items

1. **Confirm CSS dropdown fix** — The course search dropdown was styled with a white background inside the dark panel. A fix was applied (changed to `var(--green-mid)` background with white text). Needs visual confirmation on device.
2. **Verify end-to-end inference accuracy** — The slope % and clock position inference has been built but not fully validated against known putts. Accuracy depends on the GI color scale being consistent and the canvas sampling logic being correctly oriented. Should test on a known green (e.g. Minnesota Valley CC, Hole 1).
3. **Distance unit awareness** — The inferred distance is always in feet (pixel geometry × `meterToPixelScale × 3.28084`). The app supports a feet/metres toggle via `useUnits`. The `GreenSlopePanel` currently always shows "ft" in the result panel. If the user is in metres mode, the displayed value and the value sent to the app via `onSetDistance` should be consistent with the current unit.
4. **"Make Putt" API endpoint** — Chase at GI mentioned an upcoming endpoint that returns the recommended aim path given ball + hole GPS position. This could eventually supplement or replace the ZBL aim calculation. No action needed now but worth tracking.
5. **Production credit model** — The 200 test credits are sufficient for development. A commercial conversation with GI is needed before shipping to real users.
6. **Pre-fetch for offline use** — The app is used on the golf course where connectivity can be spotty. A future enhancement could pre-fetch and cache all 18 greens for a selected course before the round starts.

### Credit Usage Reference

| API Call | Cost |
| --- | --- |
| `searchCourseGroups` | 0 |
| `getCourseGroupScorecard` | 1 |
| `getSlopeImage` | 1 |
| `getElevationImage` | 1 |

Course search is debounced (400ms) and free. Scorecard is fetched once per course (in-memory cached). Images are fetched once per hole per mode (two-layer cached).