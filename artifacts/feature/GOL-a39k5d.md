---
id: GOL-a39k5d
type: feature
title: Widget improvements — all 4 yardages + live reload fix
status: Done
---
# Widget improvements — all 4 yardages + live reload fix

## Problems
- Lock screen and large home widget showed empty state ("Open ZeroBreak to enter your yardages") even after yardages were entered — widget never refreshed because `WidgetCenter.reloadAllTimelines()` wasn't being called
- Widget only showed 2 yardages per club (full total + estimated flighted total); user wants all 4: full total, full carry, flighted total, flighted carry

## Changes

### `BagBridgePlugin.swift`
- Added `import WidgetKit`
- Call `WidgetCenter.shared.reloadAllTimelines()` immediately after writing to UserDefaults, so the widget updates as soon as yardages are saved in the app

### `BagWidgetData.swift`
- Added `fullCarry` and `flightedCarry` fields to `BagWidgetRow`
- `loadBagRows()` now populates both carry fields — uses manual pelz values when calibrated, otherwise estimates at the same % offset as the totals

### `BagWidget.swift`
- `ClubRowView` now renders all 4 yardages inline: `85/80  ~81/76` (total/carry for full swing and flighted)
- Carry values shown in smaller secondary weight so the primary numbers stay readable at a glance
- Lock screen font reduced 11→10pt to accommodate wider rows
- `previewRows` updated with realistic carry values