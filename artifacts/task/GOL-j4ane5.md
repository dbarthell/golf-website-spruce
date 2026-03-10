---
id: GOL-j4ane5
type: task
title: Back button enhancement
status: Done
---
# Back button enhancement

I don’t think we need the border around the Back button on the putt log and calibrate screens

## Implementation

Removed the `border: 1px solid rgba(255, 255, 255, 0.3)` rule from the `.back-link` class in [global.css](code://src/styles/global.css) (line 656). This border was shared by both the putt log screen ([LogPage.tsx](code://src/pages/LogPage.tsx)) and the calibrate screen ([CalibratePage.tsx](code://src/pages/CalibratePage.tsx)), so removing it from the single `.back-link` rule in `global.css` cleans up the button on both screens at once.