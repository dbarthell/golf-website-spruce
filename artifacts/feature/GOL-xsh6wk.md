---
id: GOL-xsh6wk
type: feature
title: Add logo to chrome tab / bookmark
status: Completed
---
# Add logo to chrome tab / bookmark

Right now it’s the default world

## Implementation

Generated favicon PNGs from the existing `mvcc-logo.jpg` using `sips`:
- `images/favicon-32x32.png` — 32×32 px, shown in browser tabs
- `images/favicon-16x16.png` — 16×16 px, fallback for older browsers
- `images/apple-touch-icon.png` — 180×180 px, for iOS home screen bookmarks

Added the corresponding `<link>` tags to the `<head>` of `index.html`:
```html
<link rel="icon" type="image/png" sizes="32x32" href="images/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="images/favicon-16x16.png">
<link rel="apple-touch-icon" sizes="180x180" href="images/apple-touch-icon.png">
```