---
id: GOL-g67n8z
type: feature
title: Golf Intelligence Green Map Integration
status: Done
---
# Golf Intelligence Green Map Integration

## Feature Summary

Integrates the Golf Intelligence (GI) API into the ZeroBreak putting module to surface **green slope images** and use them to **automatically infer distance and slope %** from a two-tap interface. The golfer taps the hole and their ball on the slope map; the app calculates distance from pixel geometry and infers slope % from the GI color scale. Clock position is left for the golfer to set manually afterward.

The green map lives on a second slide of a full-screen swipe carousel — the main lookup view (slide 0) and the green map (slide 1) swipe horizontally like Instagram, with live drag tracking and rubber-band edges.

---

## Implementation Plan

### What Needs to Change and Why

The original plan was just to display a slope image as a visual aid. During implementation, we went further: by reading pixel colors from the GI slope image we can infer the actual slope %, eliminating a manual input step for the golfer. The carousel replaces what was a scrollable single-page layout, giving the green map its own full-screen context without pushing the clock face and results off screen.

### Architecture Overview

```
LookupPage (swipe carousel)
  ├── Slide 0: main lookup view (stimp, distance, slope, clock, results)
  └── Slide 1: GreenSlopePanel
                  ├── useCourseSelector (state hook)
                  │       ├── golfIntelligence.ts  (API client, auth, caching)
                  │       └── localStorage          (course/hole persistence)
                  └── slopeColor.ts               (pixel color → slope %)
```

### Files Created

| File | Purpose |
| --- | --- |
| [golfIntelligence.ts](code://src/lib/golfIntelligence.ts) | GI API client — OAuth token exchange, course search, scorecard (hole IDs), slope image fetch with two-layer cache (memory + localStorage) |
| [slopeColor.ts](code://src/lib/slopeColor.ts) | Canvas pixel reader — classifies GI color scale to slope % (physics-weighted 33rd-percentile, 0.79 correction factor) |
| [useCourseSelector.ts](code://src/hooks/useCourseSelector.ts) | React state hook — debounced search, course/hole persistence, explicit image load trigger |
| [GreenSlopePanel.tsx](code://src/components/GreenSlopePanel.tsx) | Green map slide UI — course search → hole picker (horizontal scroll) → 2-tap inference (hole, ball) with SVG golf ball marker overlay |

### Files Modified

| File | Change |
| --- | --- |
| [LookupPage.tsx](code://src/pages/LookupPage.tsx) | Restructured as full-screen swipe carousel with live drag (touchmove), snap-on-release, rubber-band edges, dot indicator, and `GreenSlopePanel` on slide 1 |
| [vite.config.ts](code://vite.config.ts) | Added `/gi-api` proxy (GI API, avoids CORS) and `/cf-img` proxy (CloudFront CDN, enables canvas `getImageData`) |
| [capacitor.config.ts](code://capacitor.config.ts) | Enabled `CapacitorHttp` so native iOS fetch bypasses WKWebView CORS for canvas pixel reads |
| [global.css](code://src/styles/global.css) | Carousel layout (page-carousel, page-track, page-slide, page-nav), GreenSlopePanel styles |
| `.env.local` | `VITE_GI_API_TOKEN` and `VITE_GI_CLIENT_ID` (gitignored) |

### Key Technical Decisions

**Auth flow:** GI uses a two-step OAuth-style flow — POST the static API token to `/auth/authenticateToken` to get a short-lived Bearer access token. The client caches the access token + refresh token in memory with a 1-hour TTL.

**Credit conservation:** Slope images cost 1 credit each. The app only fetches on explicit user tap ("Load Green Map"). Fetched images are cached in two layers: in-memory `Map` and `localStorage` keyed by `endpoint:holeId` (\~1-year TTL from CloudFront URL expiry).

**Canvas CORS:** CloudFront blocks `canvas.getImageData()` cross-origin. In the browser we route the image through a Vite `/cf-img` proxy. In the Capacitor iOS native build, `CapacitorHttp` handles it at the native layer.

**Clock position — manual:** Automatic clock detection was attempted (ring sampling, PCA) but proved fundamentally unreliable. Clock position is left unset after the two-tap inference; the golfer picks it manually.

**Swipe carousel:** Live drag via direct DOM ref updates (no re-renders during drag). Rubber-band resistance at both edges (0.25× factor). Snaps with `cubic-bezier(0.4,0,0.2,1)` on release.

**Golf ball marker:** SVG overlay with dimples computed in canvas pixel space from `containerRef.getBoundingClientRect()` to match the percentage-positioned SVG coordinate system.

### Remaining Open Items

1. **Paywall / paid tier** — Green map costs real credits. Plan is to gate "Load Green Map" behind a Pro subscription (\~$4.99/mo) once payments infrastructure is in place. `isPro` flag should default to `true` during beta.
2. **Distance unit awareness** — Inferred distance is always in feet. Should respect the `useUnits` toggle.
3. **Pre-fetch for offline use** — Pre-fetch all 18 greens before a round starts for offline access.
4. **"Make Putt" API endpoint** — GI mentioned an upcoming endpoint returning recommended aim path from GPS. Worth tracking.
5. **Production credit model** — Commercial conversation with GI needed before shipping to real users (200 test credits in use now).

### Credit Usage Reference

| API Call | Cost |
| --- | --- |
| `searchCourseGroups` | 0 |
| `getCourseGroupScorecard` | 1 |
| `getSlopeImage` | 1 |

Course search is debounced (400ms) and free. Scorecard is fetched once per course (in-memory cached). Images fetched once per hole (two-layer cached).