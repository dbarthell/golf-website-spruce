---
id: GOL-w21pyg
type: feature
title: UX Enhancements
status: Done
---
# UX Enhancements

My friend say I should use Mantine and Tabler (spelling?) and React to make the app look better and be more flexible for when I get it ready to launch to the app store.

---

## Implementation Plan

### What Is This?

Your friend is right — [**Mantine**](https://mantine.dev) (UI component library) + [**Tabler Icons**](https://tabler.io/icons) + **React** is a solid, modern stack for a mobile-first PWA. Currently ZeroBreak is a two-page plain HTML/CSS/JS app with no build system. This plan migrates it to React while keeping all the existing putting math intact.

---

### Current State

| File | Role |
| --- | --- |
| [index.html](code://index.html) | Main lookup UI (stimp, distance, slope, clock face, results) |
| [log.html](code://log.html) | Calibration tool (adjust backswing factor, preview table) |
| [js/mobile.js](code://js/mobile.js) | All business logic + DOM manipulation for index.html (~880 lines) |
| [js/log.js](code://js/log.js) | Calibration logic for log.html |
| [css/mobile.css](code://css/mobile.css) | Custom design system (~870 lines of hand-rolled CSS) |
| [css/log.css](code://css/log.css) | Calibration page styles |
| [data/putting.json](code://data/putting.json) | Backswing + ZBL lookup tables (the core data) |
| [sw.js](code://sw.js) | Service worker for offline/PWA |
| [manifest.json](code://manifest.json) | PWA manifest |

The app has no `package.json`, no build step, and no framework. Everything is served as static files.

---

### What Needs to Change

#### 1. Set Up a Build System

Introduce **Vite** as the build tool — it's the recommended pairing with React and Mantine, produces fast PWA builds, and has great mobile/PWA plugins.

New root files needed:

- `package.json` (React, Mantine v7, `@tabler/icons-react`, Vite, `vite-plugin-pwa`)
- `vite.config.ts`
- `tsconfig.json` (optional but recommended — see open questions)
- `index.html` (Vite entry point, replaces current one)

#### 2. Restructure as a React App

Convert the two HTML pages into React routes using **React Router**:

- `/` → main Quick Lookup screen (current `index.html`)
- `/calibrate` → Calibration screen (current `log.html`)

New `src/` directory structure:

```
src/
  main.tsx              # App entry, MantineProvider, Router
  App.tsx               # Route definitions
  pages/
    LookupPage.tsx      # Main screen
    CalibratePage.tsx   # Calibration screen
  components/
    Header.tsx
    StimpToggle.tsx
    DistanceInput.tsx
    SlopeInput.tsx
    ClockFace.tsx       # The SVG clock — most complex component
    LookupResult.tsx
    CommonDistances.tsx
    BackswingTable.tsx
    OnboardingModal.tsx
  hooks/
    usePuttingData.ts   # Fetch + cache data/putting.json
    useCalibration.ts   # localStorage calibration factor
    useLookupState.ts   # Distance, slope, stimp, clock state
  lib/
    zbl.ts              # calculateZBLVector, aimPoint, fmtInches
    backswing.ts        # findBackswingData, backswingDisplay
    interpolation.ts    # lerp, parseCell, formatVariance
    clockMath.ts        # CLOCK_DATA, getClockFactors
```

#### 3. Replace Hand-Rolled CSS with Mantine

Mantine provides a complete design system — most of the custom CSS in `mobile.css` can be replaced:

| Current CSS concept | Mantine equivalent |
| --- | --- |
| `.stimp-btn` toggle group | `SegmentedControl` or `Button.Group` |
| `.input-btn` +/- stepper | `NumberInput` with custom stepper |
| `.distance-card` grid | `SimpleGrid` + `Card` |
| `.lookup-result` panel | `Paper` |
| `.quick-table` | `Table` |
| Onboarding overlay | `Modal` with slides, or `Stepper` |
| Color variables (navy palette) | Mantine custom theme with `primaryColor` |

The navy/blue color palette (`--green-dark: #0e1b3d`, etc.) gets defined once in a **Mantine theme** and applied everywhere automatically.

#### 4. Add Tabler Icons

Replace text-only labels with icons where appropriate:

- `IconArrowLeft` for the back button on Calibrate
- `IconAdjustments` for calibration
- `IconGolf` or `IconTarget` for the app header
- `IconChevronUp` / `IconChevronDown` for the +/- stepper buttons
- `IconClock` for the clock position section label

#### 5. Keep the Clock Face as a Custom Component

The interactive clock face with SVG annotations is custom enough that it won't map to a Mantine component. Port it as a self-contained `ClockFace.tsx` React component, managing its own SVG drawing via React state + refs (instead of direct DOM manipulation).

#### 6. Preserve PWA Functionality

Use `vite-plugin-pwa` to regenerate the service worker and manifest automatically from Vite config. The existing `sw.js` and `manifest.json` are replaced by plugin output.

---

### Implementation Sequence

1. **Bootstrap the React/Vite project** in the existing repo — add `package.json`, Vite config, TypeScript config, and `src/main.tsx`
2. **Extract the business logic** from `mobile.js` into typed `src/lib/*.ts` modules (pure functions, no DOM)
3. **Build&#32;`usePuttingData`&#32;and&#32;`useCalibration`&#32;hooks** to replace the global state pattern
4. **Port&#32;`LookupPage`** — start with a working but unstyled React version of the main screen
5. **Apply Mantine theming** — configure the custom navy theme, replace CSS classes with Mantine components
6. **Port&#32;`ClockFace`** — convert the imperative DOM + SVG code to a controlled React component
7. **Port&#32;`CalibratePage`** — simpler; mostly inputs and a preview table
8. **Add Tabler Icons** — swap in icon components
9. **Port the Onboarding flow** — `OnboardingModal` using Mantine's `Modal` + `Stepper`
10. **Configure PWA** — `vite-plugin-pwa` with existing manifest metadata, update `sw.js` caching strategy
11. **Remove old files** — delete `index.html`, `log.html`, `js/`, `css/` once the React build covers them

---

### Open Questions for Human Review

Before starting, please decide on the following:

1. **TypeScript or plain JavaScript?** Mantine and Tabler Icons are both TypeScript-first. Recommend TypeScript for the new code — but if you'd rather stay in `.jsx`, that works too.
2. **App store target?** You mentioned "ready to launch to the app store." Do you mean:

- **iOS App Store / Google Play** via a native wrapper like **Capacitor** (converts the web app into a native shell)? — This affects how we handle the PWA manifest and native APIs.
- Or just submitting the **PWA itself** to the stores (Google Play supports PWAs via Trusted Web Activity; App Store PWA support is more limited)?

3. **Mantine version?** Mantine v7 is the current major version (v6 had a significant breaking change). Assuming v7 unless you have a preference.
4. **Design: how much of the current look to keep?** The current app has a polished custom look (navy gradient header, the clock face aesthetic). Options:

- Keep the current visual design but swap in Mantine components underneath (more work)
- Use Mantine defaults and let the design evolve (faster, different look)

5. **Routing strategy:** React Router is the standard choice. Alternative is to use **Next.js** instead of Vite — it adds file-based routing and server-side rendering, but that's overkill for a PWA with two screens. Sticking with Vite + React Router is recommended unless you have a reason for Next.js.
6. **Data fetching:** Currently `putting.json` is fetched at runtime. Should it be:

- Bundled directly into the app at build time (simpler, faster, works 100% offline)?
- Still fetched at runtime (allows updating the data without a new app build)?

---

## Implementation Summary

The full React + Vite + Mantine + Tabler Icons migration has been implemented. All open questions were resolved as follows:

- **TypeScript** — used throughout (`.tsx`/`.ts`)
- **Data** — bundled via `import` for full offline support (simplest, fastest; rebuild is trivial for a GitHub Pages deploy)
- **Routing** — `HashRouter` (`/#/` and `/#/calibrate`) for GitHub Pages compatibility
- **Design** — existing navy visual design preserved in `src/styles/global.css`; Mantine adds the theme system and component structure on top

### New files created

| Path | Role |
| --- | --- |
| [package.json](code://package.json) | React 18, Mantine v7, Tabler Icons, Vite 5, vite-plugin-pwa |
| [vite.config.ts](code://vite.config.ts) | Vite config — base `/golf-website/`, static copy of `images/`, PWA plugin |
| [tsconfig.json](code://tsconfig.json) | TypeScript strict mode config |
| [src/vite-env.d.ts](code://src/vite-env.d.ts) | Vite env types |
| [src/main.tsx](code://src/main.tsx) | App entry — MantineProvider + HashRouter |
| [src/App.tsx](code://src/App.tsx) | Route definitions (`/` and `/calibrate`) |
| [src/theme.ts](code://src/theme.ts) | Custom Mantine navy theme (mapped from existing CSS palette) |
| [src/styles/global.css](code://src/styles/global.css) | Full visual design preserved from mobile.css + log.css |
| [src/lib/types.ts](code://src/lib/types.ts) | TypeScript interfaces (`LagRow`, `BrysonRow`, `PuttingData`, `Calibration`) |
| [src/lib/interpolation.ts](code://src/lib/interpolation.ts) | `lerp`, `parseCell`, `formatVariance` |
| [src/lib/clockMath.ts](code://src/lib/clockMath.ts) | `CLOCK_DATA`, `getClockFactors`, `HOUR_POSITIONS` |
| [src/lib/zbl.ts](code://src/lib/zbl.ts) | `calculateZBLVector`, `fmtInches`, `aimPoint` |
| [src/lib/backswing.ts](code://src/lib/backswing.ts) | `findBackswingData`, `backswingDisplay`, `findBackswingInches` |
| [src/hooks/usePuttingData.ts](code://src/hooks/usePuttingData.ts) | Bundled JSON data (offline-first) |
| [src/hooks/useCalibration.ts](code://src/hooks/useCalibration.ts) | localStorage calibration state |
| [src/hooks/useLookupState.ts](code://src/hooks/useLookupState.ts) | Distance, slope, stimp, clock state with localStorage persistence |
| [src/components/StimpToggle.tsx](code://src/components/StimpToggle.tsx) | Stimp selector (dark + light variants) |
| [src/components/DistanceInput.tsx](code://src/components/DistanceInput.tsx) | Distance +/− with `IconChevronUp`/`IconChevronDown` |
| [src/components/SlopeInput.tsx](code://src/components/SlopeInput.tsx) | Slope +/− with `IconChevronUp`/`IconChevronDown` |
| [src/components/ClockFace.tsx](code://src/components/ClockFace.tsx) | SVG clock face with interactive buttons, hand, and ZBL annotation overlay |
| [src/components/LookupResult.tsx](code://src/components/LookupResult.tsx) | Result panel (straight mode + clock mode) |
| [src/components/CommonDistances.tsx](code://src/components/CommonDistances.tsx) | Distance grid with backswing shortcuts |
| [src/components/BackswingTable.tsx](code://src/components/BackswingTable.tsx) | Backswing reference table |
| [src/components/OnboardingModal.tsx](code://src/components/OnboardingModal.tsx) | 3-slide onboarding using `IconClock` |
| [src/pages/LookupPage.tsx](code://src/pages/LookupPage.tsx) | Main screen — all lookup logic + `IconAdjustments` calibrate link |
| [src/pages/CalibratePage.tsx](code://src/pages/CalibratePage.tsx) | Calibration screen — `IconArrowLeft` back, full preview table |

### Modified files

| Path | Change |
| --- | --- |
| [index.html](code://index.html) | Replaced with Vite entry point (`<div id="root">` + `<script type="module">`) |

### Verified

- `npx tsc --noEmit` — passes with zero errors (strict mode)
- `npx vite build` — clean production build (220 kB JS, 218 kB CSS gzipped to 70 + 32 kB)
- Dev server starts at `http://localhost:5173/golf-website/`
- PWA service worker + manifest auto-generated by `vite-plugin-pwa`
- `images/` copied to `dist/images/` by `vite-plugin-static-copy`

### Notes

- `log.html`, `js/`, `css/` old files remain on disk but are no longer served (per step 11 of the plan — remove once the React build is confirmed in production)
- `data/putting.json` stays in place (still used by build-time JSON import)

---

## Mantine Components Pass

All six UI components that were previously using hand-rolled CSS buttons/divs have been replaced with proper Mantine components.

### Component replacements

| Component | Before | After |
| --- | --- | --- |
| [StimpToggle.tsx](code://src/components/StimpToggle.tsx) | Custom `<button>` group | `SegmentedControl` |
| [DistanceInput.tsx](code://src/components/DistanceInput.tsx) | Custom `<input>` + `<button>` | `NumberInput` + `ActionIcon` + `Group` |
| [SlopeInput.tsx](code://src/components/SlopeInput.tsx) | Custom `<input>` + `<button>` | `NumberInput` + `ActionIcon` + `Group` |
| [CommonDistances.tsx](code://src/components/CommonDistances.tsx) | Custom `<button>` in CSS grid | `SimpleGrid` + `UnstyledButton` |
| [LookupResult.tsx](code://src/components/LookupResult.tsx) | Plain `<div>` | `Paper` |
| [BackswingTable.tsx](code://src/components/BackswingTable.tsx) | Plain `<table>` | Mantine `Table` / `Table.Thead` / `Table.Tr` / `Table.Th` / `Table.Td` |
| [OnboardingModal.tsx](code://src/components/OnboardingModal.tsx) | Custom `position:fixed` overlay | `Modal` (fullScreen) + `Button` |

### Theme update

[theme.ts](code://src/theme.ts) — added custom `breakpoints` to match the app's phone-first layout:

- `xs: 23.4375em` (375 px — iPhone SE / 12 / 13 / 14 / 15)
- `sm: 30em` (480 px)

This lets `SimpleGrid cols={{ base: 2, xs: 3, sm: 4 }}` exactly reproduce the previous CSS media-query grid breakpoints.

### CSS additions

[global.css](code://src/styles/global.css) — added overrides at the bottom for Mantine component internals:

- `.stimp-sc-dark` / `.stimp-sc-indicator-dark` / `.stimp-sc-label-dark[data-active]` — SegmentedControl dark variant
- `.stimp-sc-light` / `.stimp-sc-indicator-light` / `.stimp-sc-label-light[data-active]` — SegmentedControl light variant
- `.onboarding-modal-content` / `.onboarding-modal-body` — Modal fullscreen navy background + flex layout

### Verified

- `npx tsc --noEmit` — zero errors (strict mode)
- `npx vite build` — clean build (338 kB JS / 219 kB CSS, gzipped 106 + 32 kB)