---
id: GOL-mggd7c
type: feature
title: Logo and clock update
status: Done
---
# Logo and clock update

Want to 1. make the background green behind the logo letters the exact color as the header and 2) show the clock numbers above the straight line and dotted line. Right now they obscure the numbers

## Implementation

### 1. Logo background color ([global.css](code://src/styles/global.css))
Added `background-color: var(--green-dark)` to `.header-logo`. This ensures any area visible behind the logo image (particularly at the rounded corners created by `border-radius: 24%`) exactly matches the header's dark green (`#013c36`), making the logo blend seamlessly into the header gradient.

### 2. Clock numbers above annotation lines ([ClockFace.tsx](code://src/components/ClockFace.tsx))
The SVG annotation overlay (straight measurement line and dotted break line) was rendered **after** the hour buttons in the DOM, so it sat visually on top of them and obscured the clock numbers. Fixed by reordering: the `<ClockSVG>` annotation is now rendered **before** the hour buttons, so the buttons always appear on top of the lines.