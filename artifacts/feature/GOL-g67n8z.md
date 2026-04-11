---
id: GOL-g67n8z
type: feature
title: Golf Intelligence API
status: In Progress
---
# Golf Intelligence API

I want to explore Golf Intelligence's API for my app: "Hey Dan,

Sorry for not getting back to you on this.. your messages were stuck on one of my filters.

I've created a test account for you in our GolfIntelligence console.

You can log in here: [https://console.golfintelligence.com/](https://console.golfintelligence.com/)

- Login: Your email address
- Password: The verification code sent to your inbox

Once logged in, you'll find:

- **Postman Collections** with ready-to-use requests for all our API endpoints
- **Swagger documentation** (interactive API reference)
- Your **Active Token** and **Client ID** for authentication

You currently have **200 test credits**, which should be enough to download several full courses, render green slope images, elevations, and more.

Let me know if you need more credits, want me to add other team members, or have any questions about the endpoints or setup.

Happy to help!

Best, Chase" 

"

[
Snapshot](https://console.golfintelligence.com/partner/snapshot)[
API Account](https://console.golfintelligence.com/partner/info)[
Credits](https://console.golfintelligence.com/partner/credits)[
Live Log](https://console.golfintelligence.com/partner/live)[
Log History](https://console.golfintelligence.com/partner/log)

Account Info

| Account | Coniferous.dev |
| --- | --- |
| Created | 04/10/2026 |
| ClientId | 1029290 |
| Credit Balance | 200 |
| Balanced Updated | 04/10 |
| Active Token | b4f8f9a5-f7c3-459d-8eee-a17f76ba85cf |
| Token Expires | 04/10/2027 |
| Client Login | Coniferousdev@GolfIntelligence.com |

Token

| Token | Created | Expires | IsActive |
| --- | --- | --- | --- |
| b4f8f9a5-f7c3-459d-8eee-a17f76ba85cf | 04/10/2026 | 04/10/2027 | YES |

[Reset Token]()

API Access

| Access Type | Access | Cost |
| --- | --- | --- |
| **Render Green Slope** /greens/getSlopeImage | YES | 1 |
| **Render Green Elevation** /greens/getElevationImage | YES | 1 |
| **Full Course Detail w/ Scorecard & GPS** /courses/getCourseGroupDetail | YES | 3 |
| **Course Search** /courses/searchCourseGroups | YES | 0 |
| **Course ScoreCard** /courses/getCourseGroupScorecard | YES | 1 |
| **Course GPS** /courses/getCourseGroupGPS | YES | 2 |

| API Region | Access |
| --- | --- |
| **Minor Countries** | YES |
| **Asia** | YES |
| **Australia & New Zealand** | YES |
| **Mainland Europe** | YES |
| **United Kingdom** | YES |
| **Canada** | YES |
| **United States** | YES |

Account Admin

| GolferId | Name | Created |
| --- | --- | --- |
| 1029291 | null null | 04/10/2026 |

"

also check my gmail thread (dan@coniferous.dev) with Chase at Golf Intelligence for more context. I specifically want to their green slope data in the putting module of the app

---

## Implementation Plan

### Summary

Integrate the Golf Intelligence API into the ZeroBreak putting module to surface **green slope images** (and optionally elevation images) on the `LookupPage`. The primary goal is to let a golfer select their course and hole number so they can see a visual slope map of the green while they're filling out the slope/stimp inputs — giving them a real reference image instead of relying on eyeballing.

The API is image-based: `/greens/getSlopeImage` returns a rendered slope image for a given green (1 credit each). There is no endpoint that returns a raw slope percentage to auto-fill the input — so the integration is a **visual aid**, not an auto-populate. The golfer looks at the image and sets the slope manually.

### What the API Gives Us

- **Course Search** (`/courses/searchCourseGroups`) — free, find a course by name
- **Green Slope Image** (`/greens/getSlopeImage`) — 1 credit, returns a rendered slope overlay image keyed by course + hole
- **Green Elevation Image** (`/greens/getElevationImage`) — 1 credit, 3D elevation view of a green
- **"Make Putt" feature** (coming soon per Chase) — given ball & hole position, returns aim path calculation. This could eventually replace or augment the ZBL math.

### Files to Create

| File | Purpose |
|---|---|
| [src/lib/golfIntelligence.ts](code://src/lib/golfIntelligence.ts) | API client — `searchCourses()`, `getSlopeImage()`, `getElevationImage()` |
| [src/hooks/useCourseSelector.ts](code://src/hooks/useCourseSelector.ts) | State hook — selected course, hole, cached image URLs |
| [src/components/GreenSlopePanel.tsx](code://src/components/GreenSlopePanel.tsx) | UI panel — course search, hole picker, slope/elevation image display |

### Files to Modify

| File | Change |
|---|---|
| [src/pages/LookupPage.tsx](code://src/pages/LookupPage.tsx) | Add `<GreenSlopePanel>` below the slope/stimp inputs, above the clock face |
| [src/hooks/useLookupState.ts](code://src/hooks/useLookupState.ts) | Persist selected course + hole to localStorage so it survives navigation |

### Implementation Steps

**Step 1 — API Client (`src/lib/golfIntelligence.ts`)**

Create a thin fetch wrapper. Auth is a Bearer token in the `Authorization` header plus `ClientId` query param (need to confirm exact auth shape from Swagger — see open questions below).

```
searchCourseGroups(query: string): Promise<CourseGroup[]>
getSlopeImage(courseGroupId: string, hole: number): Promise<string>  // returns image URL or blob URL
getElevationImage(courseGroupId: string, hole: number): Promise<string>
```

Credentials stored in environment variables (`VITE_GI_TOKEN`, `VITE_GI_CLIENT_ID`) — **never committed to source**. For the iOS Capacitor build these get baked in at build time via Vite's `import.meta.env`.

**Step 2 — Course Selector Hook (`src/hooks/useCourseSelector.ts`)**

- `courseQuery` + `setCourseQuery` — search string
- `searchResults` — debounced results from `searchCourseGroups`
- `selectedCourse` / `setSelectedCourse` — persisted to localStorage
- `selectedHole` / `setSelectedHole` — 1–18, persisted
- `slopeImageUrl` — fetched on demand, cached by `courseId+hole` key so we don't re-spend credits

**Step 3 — Green Slope Panel (`src/components/GreenSlopePanel.tsx`)**

Collapsible panel (closed by default to keep the main UI clean):
1. Search box → typeahead results → tap to select course
2. Hole number picker (1–18 chips or stepper)
3. "Load Green Map" button (with credit cost warning: "1 credit")
4. Displays slope image full-width below; toggle to elevation image

**Step 4 — Wire into LookupPage**

Add `<GreenSlopePanel>` between the inputs section and the `<WalkOffTip>` / clock face. Collapsed by default.

**Step 5 — Environment / Build config**

Add `VITE_GI_TOKEN` and `VITE_GI_CLIENT_ID` to `.env.local` (gitignored) and document in `README.md`.

### Credit Usage Strategy

- Course search is free — safe to call on every keystroke (debounced)
- Slope/elevation images cost 1 credit each — only fetch on explicit user tap ("Load Green Map"), not automatically
- Cache fetched image URLs in memory (and optionally `sessionStorage`) so navigating away and back doesn't re-charge credits
- 200 test credits → up to 200 unique green maps before needing a top-up

### Open Questions / Decisions Needed

1. **Auth format** — The console shows a Bearer token + ClientId, but I need to confirm the exact header/param names from the Swagger docs or Postman collection. Needs a quick check before writing the API client.
2. **Endpoint for getSlopeImage** — What identifies a specific green? Is it `courseGroupId` + `holeNumber`, or is there a separate `greenId` lookup step? (Dan asked this in the email thread; Chase's account setup reply didn't answer it directly.)
3. **Image format returned** — Does the endpoint return a redirect to an image URL, a base64 blob, or a binary stream? Affects how we display it.
4. **"Make Putt" ETA** — Chase mentioned this feature is coming soon. If it returns an aim path given ball + hole position, it could eventually replace the ZBL aim math in the app. Worth asking Chase for a timeline so we know whether to architect for it now.
5. **Credit model for production** — 200 test credits is fine for dev/QA. What's the pricing for production usage? Per-credit, subscription? Need a commercial conversation before shipping to real users.
6. **Offline/on-course use** — The app is used on the golf course where connectivity may be poor. Should we pre-fetch and cache green images for a selected course before the round, or always fetch on-demand?