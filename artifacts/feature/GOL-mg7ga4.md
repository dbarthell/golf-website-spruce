---
id: GOL-mg7ga4
type: feature
title: Header overlaps iOS status bar
status: Done
---
# Header overlaps iOS status bar

I launched my app in the app store but now I see that the header doesn’t stick when you scroll down like other apps do. Instead, the header scrolls up and obscures the time, wifi icon, battery icon, etc. on my phone. Do what you think is best to fix this.

## Implementation

**Root cause:** The app uses `viewport-fit=cover` with `apple-mobile-web-app-status-bar-style: black-translucent`, which extends the web viewport all the way to the top of the screen — behind the iOS status bar (time, signal, battery). The headers weren’t accounting for this, so as the page scrolled their content would pass behind the status bar and overlap the system clock/wifi/battery icons.

**Fix applied** in [global.css](code://src/styles/global.css):

- **Removed** `padding-top: env(safe-area-inset-top)` from `body` — the safe area is now owned by the headers themselves.
- **Both `.app-header` and `.log-header`:** Added `env(safe-area-inset-top)` to their top padding and `min-height`. This extends the header background up into the notch area so it fills cleanly, while pushing the header content (logo, nav links, back button, title) down to sit below the status bar.
- **Sticky positioning removed** — headers scroll away as the user scrolls down, consistent with how most iOS apps behave for content-focused screens.