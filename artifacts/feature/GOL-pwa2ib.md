---
id: GOL-pwa2ib
type: feature
title: PWA Install Banner
status: Backlog
priority: Medium
size: S
tasks: []
---
# PWA Install Banner

Add a dismissible in-app install banner to `LookupPage` that appears when the browser fires the `beforeinstallprompt` event. The app already ships with `manifest.json` and `sw.js`; this feature makes the install path discoverable.

## Background

Most users never find the browser's native "Add to Home Screen" option. An explicit in-app prompt dramatically increases install rates. A PWA putt tool on the home screen opens faster on the course.

## Acceptance Criteria

- A new `useInstallPrompt` hook captures the `beforeinstallprompt` event and exposes `{ canInstall, prompt, dismiss }`
- A slim `InstallBanner` component appears at the top of `LookupPage` only when `canInstall && !dismissed`
- Tapping the banner's install button triggers the native install dialog
- Tapping dismiss hides the banner and persists the dismissed state to `localStorage` so it doesn't reappear on reload
- No banner is shown on browsers that don't support `beforeinstallprompt` (e.g., Safari, Firefox)

## Key Files

- New: [useInstallPrompt.ts](code://src/hooks/useInstallPrompt.ts)
- New: [InstallBanner.tsx](code://src/components/InstallBanner.tsx)
- Edit: [LookupPage.tsx](code://src/pages/LookupPage.tsx) — render `<InstallBanner>` at top of page
