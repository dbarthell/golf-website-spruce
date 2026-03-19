---
id: GOL-mczg93
type: feature
title: Launch in iOS App Store
status: Done
---
# Launch in iOS App Store

Need to prepare to launch my app in the iOS store now that I have an Apple Developer Account and a DUNS number. Help me prepare the code and a checklist to launch.

---

## Implementation Plan

### Summary

ZeroBreak is a React + TypeScript + Vite PWA currently deployed to GitHub Pages at the `/golf-website/` subpath. To get it into the iOS App Store, the best approach is to wrap the existing web app with **Capacitor** (by Ionic), which creates a native iOS shell around the web content. This avoids a full rewrite while producing a genuine `.ipa` that Apple accepts.

The main code changes are:

1. Conditionally switch the Vite base path so GitHub Pages and Capacitor can coexist
2. Install and configure Capacitor with a proper Bundle ID
3. Generate App Store–quality app icons and splash screens (1024×1024 required, **no transparency**)
4. Fix viewport/safe-area insets so the UI doesn't get clipped under the iPhone notch/home indicator
5. Host a privacy policy page (Apple hard requirement)
6. Initialize the iOS Xcode project, configure signing, and submit

---

### Files That Need to Change

| File | What Changes |
| --- | --- |
| [vite.config.ts](code://vite.config.ts) | Use `CAPACITOR_BUILD` env var to switch `base` and PWA manifest paths between `'/'` and `'/golf-website/'` |
| [manifest.json](code://manifest.json) | Update `start_url` and `scope` to `'/'` (used by the native build only) |
| [index.html](code://index.html) | Add `viewport-fit=cover` to meta viewport; update icon hrefs after icon regeneration |
| [sw.js](code://sw.js) | Delete — legacy file superseded by `vite-plugin-pwa`; hardcoded `/golf-website/` paths are stale |
| `capacitor.config.ts` *(new)* | New file: defines `appId`, `appName`, `webDir: 'dist'` |
| `package.json` | Add Capacitor deps; add `build:ios` script using `cross-env` |
| `public/privacy.html` *(new)* | Simple privacy policy page hosted on GitHub Pages |
| CSS (global or layout) | Add `padding-top: env(safe-area-inset-top)` and `padding-bottom: env(safe-area-inset-bottom)` to the app shell |

---

### Step-by-Step Implementation

#### Phase 1 — Conditional Base Path (Keep GitHub Pages Alive)

The key insight: hardcoding `base: '/'` would immediately break the GitHub Pages deployment. Instead, use an environment variable to switch at build time.

**Step 1a: Update&#32;**[vite.config.ts](code://vite.config.ts)

```ts
const isCapacitor = process.env.CAPACITOR_BUILD === 'true';
const base = isCapacitor ? '/' : '/golf-website/';

export default defineConfig({
  base,
  plugins: [
    react(),
    viteStaticCopy({ targets: [{ src: 'images', dest: '.' }] }),
    VitePWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'ZeroBreak',
        short_name: 'ZeroBreak',
        start_url: isCapacitor ? '/' : '/golf-website/',
        scope: isCapacitor ? '/' : '/golf-website/',
        display: 'standalone',
        background_color: '#0e1b3d',
        theme_color: '#0e1b3d',
        icons: [
          { src: 'images/logo.png', sizes: 'any', type: 'image/png' },
        ],
      },
    }),
  ],
});
```

**Step 1b: Update&#32;**[package.json](code://package.json)**&#32;scripts**

```json
"build": "tsc && vite build",
"build:ios": "tsc && cross-env CAPACITOR_BUILD=true vite build && npx cap sync ios"
```

Install `cross-env` (makes env vars work on Windows terminals too):

```bash
npm install -D cross-env
```

#### Phase 2 — Install Capacitor

```bash
npm install @capacitor/core @capacitor/cli @capacitor/ios
npx cap init ZeroBreak com.YOURNAME.zerobreak --web-dir dist
npx cap add ios
```

> ⚠️ The `appId` you set here is **permanent** — it can never be changed after the app is live on the App Store. Choose carefully (see Open Questions).

**Create&#32;`capacitor.config.ts`:**

```ts
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.YOURNAME.zerobreak',
  appName: 'ZeroBreak',
  webDir: 'dist',
};

export default config;
```

#### Phase 3 — Fix Viewport Safe Areas (Notch / Home Indicator)

iPhones clip web content under the status bar notch and home indicator unless you explicitly opt in to the full screen and add padding.

**Step 3a: Update&#32;**[index.html](code://index.html)**&#32;meta viewport**

Change:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

To:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
```

**Step 3b: Add safe-area padding in CSS**

In the app's global CSS or the top-level layout component, add:

```css
#root {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
}
```

This ensures the Onboarding Modal and bottom inputs don't sit behind the iPhone clock/battery icons or the home indicator bar.

#### Phase 4 — App Icons & Splash Screen

The App Store requires a **1024×1024 PNG** icon with **zero transparency** — Apple rejects binaries with transparent pixels. Also generate a splash screen that matches the dark navy theme (`#0e1b3d`) to avoid a jarring white flash on launch.

**Step 4a: Prepare source assets**

Place these files in an `assets/` folder in the project root:

- `assets/icon.png` — 1024×1024 PNG, no transparency, your ZeroBreak logo
- `assets/splash.png` — 2732×2732 PNG, dark navy background with centered logo

> ❓ **Open Question (icon):** Is the existing `logo.png` / `pwa-icon.png` high-res and transparency-free? If either file uses transparent corners, the App Store upload will fail. May need a new export from original source.

**Step 4b: Generate all sizes with&#32;`@capacitor/assets`**

```bash
npm install -D @capacitor/assets
npx capacitor-assets generate --ios
```

This writes all required icon sizes (20pt through 1024pt) and splash screen variants directly into the Xcode project under `ios/App/App/Assets.xcassets/`.

#### Phase 5 — Privacy Policy Page

Apple **will reject the binary** if no privacy policy URL is provided, even if the app collects zero data.

**Create&#32;`public/privacy.html`** (served at `https://yourusername.github.io/golf-website/privacy.html`):

```html
<!DOCTYPE html>
<html>
<head><title>ZeroBreak Privacy Policy</title></head>
<body>
  <h1>Privacy Policy</h1>
  <p>ZeroBreak does not collect, store, or transmit any personal data.
  All putting calculations happen locally on your device.
  No accounts, no analytics, no third-party tracking.</p>
  <p>Last updated: March 2026</p>
</body>
</html>
```

The URL to submit to App Store Connect: `https://yourusername.github.io/golf-website/privacy.html`

#### Phase 6 — iOS Project Configuration in Xcode

```bash
npm run build:ios
npx cap open ios
```

Open the `.xcworkspace` file (not `.xcodeproj`). In Xcode:

- **Bundle Identifier:** Must match `capacitor.config.ts` `appId` exactly
- **Team (Signing):** Select your Apple Developer account
- **Deployment Target:** iOS 16+ recommended (iOS 15 is fine too; covers \~98% of devices)
- **Version:** `1.0`
- **Build:** `1`
- **Display Name:** `ZeroBreak`
- **Supported Orientations:** Portrait only (uncheck Landscape Left / Landscape Right)

#### Phase 7 — App Store Connect Listing

In [App Store Connect](https://appstoreconnect.apple.com), create a new app and fill in:

| Field | Value |
| --- | --- |
| App Name | ZeroBreak |
| Subtitle | Putt Break Calculator |
| Primary Category | Sports |
| Description | Describe the physics-based green-reading tool for golfers |
| Keywords | putting, break, golf, slope, green reading, caddie, stimp, physics (100 chars max) |
| Privacy Policy URL | `https://yourusername.github.io/golf-website/privacy.html` |
| Age Rating | 4+ |
| Screenshots | Required: iPhone 6.5" (14 Pro Max). Optional: iPad |

> 💡 **Keywords tip:** Avoid trademarked methodology names (e.g., "AimPoint") in the description text. Use "scientific putting" and "physics-based" instead.

#### Phase 8 — Archive & Submit

```bash
npm run build:ios   # builds dist/ and syncs to ios/
npx cap open ios    # opens Xcode
```

In Xcode:

1. Select "Any iOS Device (arm64)" as the build target (not a simulator)
2. **Product → Archive**
3. In the Organizer: **Distribute App → App Store Connect → Upload**
4. In App Store Connect: select the uploaded build, complete the submission form, **Submit for Review**

Apple review typically takes 1–3 business days.

---

### Launch Checklist

- [x] `vite.config.ts` updated with conditional `CAPACITOR_BUILD` base path
- [x] `cross-env` added to devDependencies; `build:ios` script added to `package.json`
- [x] `capacitor.config.ts` created (`appId: com.zerobreakgolf.zerobreak`)
- [x] `viewport-fit=cover` added to [index.html](code://index.html) meta viewport
- [x] Safe-area inset padding added to [global.css](code://src/styles/global.css)
- [x] `public/privacy.html` created (will deploy to GitHub Pages with next push)
- [x] Legacy [sw.js](code://sw.js) deleted
- [x] `npm install` — cross-env and @capacitor/assets installed
- [x] iOS project added: `npx cap add ios`
- [x] 2732×2732 splash PNG generated at `assets/splash.png`
- [x] Icon generated at `assets/icon.png` (cropped from `images/zb-logo.jpg`, 1024×1024)
- [x] `npm run build:ios` succeeds
- [x] Xcode opened: Bundle Identifier confirmed as `com.zerobreakgolf.zerobreak`
- [x] Xcode: Apple ID added and Team selected in Signing & Capabilities
- [x] Xcode: orientations locked to Portrait only (General tab)
- [x] `npx capacitor-assets generate --ios` run successfully
- [x] ⏳ **Waiting on Apple:** Developer account fully validated
- [x] Register `com.zerobreakgolf.zerobreak` App ID at developer.apple.com → Identifiers
- [x] iOS Distribution certificate created and installed in Keychain
- [x] App Store provisioning profile created (ZeroBreak App Store) and selected in Xcode Release config
- [x] App Store Connect: app record created
- [x] Archive built and uploaded from Xcode
- [ ] App Store Connect: all metadata filled in
- [ ] Screenshots captured for iPhone 6.5" display
- [ ] Submitted for App Review in App Store Connect

---

### Open Questions for Human Review

1. **Bundle ID:** What permanent identifier to use? (e.g., `com.dbarthell.zerobreak`) — must match your Apple Developer account and can never be changed.
2. **Icon quality:** Does the existing logo have a high-res, transparency-free version at 1024×1024? This is the most common reason for a failed upload.
3. **App pricing:** Free, paid ($0.99?), or freemium?
4. **`sw.js`&#32;deletion:** Confirm it's safe to delete — it's a legacy file from before the Vite migration and conflicts with the `vite-plugin-pwa` service worker.