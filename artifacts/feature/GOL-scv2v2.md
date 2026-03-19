---
id: GOL-scv2v2
type: feature
title: Update logo and color pallette for app
status: In Review
---
# Update logo and color pallette for app

I want to update the in app logo, the pwa logo and the entire color palette based on the image on my desktop folder. I’d like to test it first too on my phone before deploying.

---

## Implementation Plan

### What’s Changing and Why

The app currently uses a dark navy/electric-blue color scheme (deep `#060e19` backgrounds, `#1e6bff` blue accents) paired with the old chrome "ZB" logo. The reference image on the Desktop (`8ec74106-a11c-4b79-95c3-6cfc8813f701.jpg`) introduces a new brand identity: a "7B" mark on a **deep forest green** background with a **warm gold** accent border and a white golf ball. The entire UI palette needs to shift from navy/blue → forest green/gold to match.

---

### New Color Palette

Derived from the reference logo image:

| Variable | Old Value | New Value | Role |
| --- | --- | --- | --- |
| `--green-dark` | `#060e19` (deep navy) | `#0d3d28` | Dark backgrounds, headers |
| `--green-mid` | `#0c1c35` (navy mid) | `#1a5c3c` | Gradient end, panels |
| `--green-accent` | `#1e6bff` (electric blue) | `#c9a86a` | Buttons, active states, CTAs |
| `--green-light` | `#4d8fff` (light blue) | `#3a8a5a` | Secondary accents, up-slope indicator |
| `--green-pale` | `#dce8ff` (pale blue) | `#e8f5ec` | Hover rows, outcome highlights, pale tints |

---

### Files to Modify

#### 1. `src/styles/global.css` — CSS Custom Properties

Update the 5 palette variables in `:root`. Also update the `.header-logo` box-shadow (currently a blue glow `rgba(30, 107, 255, 0.5)`) to a gold glow (`rgba(201, 168, 106, 0.5)`).

#### 2. `src/theme.ts` — Mantine Theme

Replace the `navy` color scale (currently blues) with a `forestGreen` scale to match the new palette, and rename `primaryColor` accordingly. Proposed 10-stop scale:

```
forestGreen: [
  ‘#e8f5ec’, // 0
  ‘#c5e0cf’, // 1
  ‘#9ac9b0’, // 2
  ‘#6ab090’, // 3
  ‘#3a9670’, // 4
  ‘#2d7a58’, // 5 – accent equivalent
  ‘#235f44’, // 6
  ‘#1a5c3c’, // 7 – --green-mid
  ‘#0d3d28’, // 8 – --green-dark
  ‘#07231a’, // 9
]
```

#### 3. `index.html` — Theme Color Meta Tag

Update `<meta name="theme-color" content="#10223c">` → `#0d3d28`

#### 4. `manifest.json` — PWA Manifest

Update `background_color` and `theme_color` from `#060e19` → `#0d3d28`.
Update the `icons` entry to point to the new image file (see below).

#### 5. `images/` — Logo Image Files

- **Copy** `~/Desktop/8ec74106-a11c-4b79-95c3-6cfc8813f701.jpg` into `images/` as `zb-logo-new.jpg` (in-app header) and as `pwa-icon-new.png` (PWA icon — will need to be exported as PNG at appropriate size, ideally 512×512).
- Update `src/pages/LookupPage.tsx` line 176: change `src="images/zb-logo.jpg"` → `src="images/zb-logo-new.jpg"`.
- Update `manifest.json` icon `src` to point to the new PWA icon.
- Update `index.html` `<link rel="icon">` and `<link rel="apple-touch-icon">`.

#### 6. iOS App Icon — `ios/App/App/Assets.xcassets/AppIcon.appiconset/AppIcon-512@2x.png`

The single iOS icon file needs to be replaced with the new logo image resized to 1024×1024 px (512pt @2x). This will update the icon in Xcode for the next TestFlight / App Store build.

---

### Implementation Sequence

1. **Copy & rename** the new logo image from Desktop into `images/`
2. **Update CSS variables** in `global.css` (5 color vars + `header-logo` glow)
3. **Update Mantine theme** in `theme.ts` (rename color scale, update values)
4. **Update&#32;`index.html`** theme-color meta tag
5. **Update&#32;`manifest.json`** background/theme colors + icon path
6. **Update&#32;`LookupPage.tsx`** img src to new logo filename
7. **Replace iOS app icon** PNG with the new image at 1024×1024
8. **Test on phone** — install PWA via Safari to verify the new icon and palette look correct before deploying

---

## Implementation Summary

### Images

- Copied `~/Desktop/8ec74106-a11c-4b79-95c3-6cfc8813f701.jpg` → [zb-logo-new.jpg](code://images/zb-logo-new.jpg) (in-app header logo)
- Converted + resized to 512×512 PNG → [pwa-icon-new.png](code://images/pwa-icon-new.png) (PWA / browser-tab icon)
- Replaced iOS app icon at [AppIcon-512@2x.png](code://ios/App/App/Assets.xcassets/AppIcon.appiconset/AppIcon-512@2x.png) with new image at 1024×1024 px

### Color Palette — [global.css](code://src/styles/global.css)

Updated CSS custom properties from navy/blue → forest green/gold:

- `--green-dark` → `#013c36`
- `--green-mid` → `#0a4a42`
- `--green-accent` → `#c9a86a` (warm gold, used for CTAs, log buttons, save buttons)
- `--green-light` → `#1d6b42`
- `--green-pale` → `#f0f7f3`

### Mantine Theme — [theme.ts](code://src/theme.ts)

Renamed color scale from `navy` → `forestGreen`, updated all 10 stops to match new palette.

### PWA / Meta — [index.html](code://index.html) & [manifest.json](code://manifest.json)

- `theme-color` updated to `#013c36`
- `manifest.json` `background_color` + `theme_color` updated to `#013c36`
- Icons updated to `images/pwa-icon-new.png`

### UI Redesign (Airbnb-style layout) — [LookupPage.tsx](code://src/pages/LookupPage.tsx) & [global.css](code://src/styles/global.css)

Refactored the main page from a single full-screen dark green hero into a layered layout:

- **Dark green header** (`app-header`) — sticky, 64px, dark green gradient, logo (3rem) + nav links. Consistent height across main, Log, and Calibrate pages via `min-height: 64px`.
- **White inputs panel** (`lookup-body`) — stimp toggle (green-dark active state), distance + slope inputs (grey cards on white), walkoff tip. Compact spacing.
- **Dark green clock band** (`clock-section`) — full-width dark green gradient section housing the clock face. Clock face is a transparent circle so the band background shows through naturally. SVG annotations (ZBL dot, aim line, slope indicator) render in white against the dark background.
- **White result panel** (`lookup-result-body`) — result card with gold (`--green-accent`) top accent stripe, near-black numbers for readability.
- **Grey reference sections** — backswing table and common distances both use `--gray-100` background so white table/cards float on them. Highlighted backswing row uses a deeper green (`#d8ede0`). Distance card backswing values use `--green-light` for brand colour without blending into the distance number.

### Component changes

- [StimpToggle.tsx](code://src/components/StimpToggle.tsx) — both variants now use `size="sm"` with compact label padding for consistent height
- [ClockFace.tsx](code://src/components/ClockFace.tsx) — clock face restored to 280px; button center corrected from C=140 → C=137 to account for 3px border (box-sizing: border-box makes interior 274px, true center 137px), fixing asymmetric button spacing; watchmaker-inspired radial gradient dial and gold SVG center pivot dot added
- [OnboardingModal.tsx](code://src/components/OnboardingModal.tsx) — added "Walk it off" slide (measuring distance by pacing) between backstroke and calibration slides; removed `WalkOffTip` component from main lookup page
- [LookupResult.tsx](code://src/components/LookupResult.tsx) — gold accent top border now only applied via `lookup-result--has-result` class, so empty state shows a plain card

### Calibrate & Log pages

- Distance chips and log round selector chips active state changed from `--gray-800` → `--green-dark` for consistency across all pages

### Favicon

- Created [favicon.png](code://images/favicon.png) — an 8% centre-cropped zoom of the PWA icon so ZB letters fill the browser tab at small sizes
- Removed white corner artifacts from `pwa-icon-new.png` and iOS app icon via rounded-rectangle transparency mask
- `index.html` `<link rel="icon">` now points to `favicon.png`