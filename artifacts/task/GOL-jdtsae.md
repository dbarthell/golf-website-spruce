---
id: GOL-jdtsae
type: task
title: View pane moves
status: done
---
# View pane moves

Sometimes when I press the plus button for the distance the view pane moves to the left exposing a white right margin. it doesn’t happen on the other side

## Work Done

Fixed in `css/mobile.css` with three targeted changes:

1. **`overscroll-behavior-x: none` on `html`** — Prevents the browser from triggering a horizontal overscroll/bounce when tapping near the right edge of the screen. This was the primary cause: tapping the `+` button (right side) would momentarily shift the visual viewport left, exposing the white background behind the content.

2. **`position: relative` on `body`** — Without a positioned ancestor, `overflow-x: hidden` on `body` is enforced by the viewport rather than the element, and can be bypassed by touch-based horizontal scroll. Adding `position: relative` makes the overflow clipping context explicit and reliable.

3. **`touch-action: manipulation` on `.input-btn`** — Tells the browser these buttons only respond to tap/manipulation, not pan gestures. This prevents the browser from misinterpreting a tap near the edge as a horizontal swipe, and also removes the 300ms tap delay for a snappier feel.