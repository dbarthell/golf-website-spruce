---
id: GOL-tr1t71
type: task
title: Swing thoughts
---
# Swing thoughts

Add swing thoughts (subject to change) to the placeholder on the website. I saved of photo of them on my desktop.

## Implementation Plan

### Photo Transcription (`~/Desktop/IMG_5308.heic`)

I read the handwritten notes from the photo. The page has three sections:

**Swing:**
1. Rotate like Bryson
2. Stretch w/ arms + hands
3. Right heel + hip initiate
4. Head still
5. Turn head to right at top of swing
6. Shaft straight up at address
7. Head down at top to prevent heel contact *(starred)*

**Short Game:**
1. Pivot of body
2. Throw w/ right arm to gauge distance
3. Read putt from side; front first feel
4. Speed in bunkers; use full swing
5. Aimpoint from center of cup
6. Accelerate putter like Bernhard
7. *(Two starred items at the bottom are hard to read — something about feet still / hip / pyramid — please verify)*

**Putter Notes (below red margin):**
- Ruler drill + planer slope system *(starred)*
- 9/step like Bryson; look for straight putt
- 3 feet ... w/ straight putt ... inches *(partially illegible)*

> **Note:** A few lines are difficult to decipher from the handwriting. The items above flagged as unclear should be verified before going live.

---

### What Changes

#### 1. Replace current Swing Thoughts content (`index.html`, lines 82-87)

The current placeholder has 4 generic bullet points:
```
- Smooth tempo — 3:1 backswing to downswing ratio
- Complete the shoulder turn
- Trust the swing, commit to the shot
- Weight forward at impact
```

**Replace with** the 7 swing thoughts from the photo:
```html
<ul>
  <li>Rotate like Bryson</li>
  <li>Stretch w/ arms + hands</li>
  <li>Right heel + hip initiate</li>
  <li>Head still</li>
  <li>Turn head to right at top of swing</li>
  <li>Shaft straight up at address</li>
  <li><strong>Head down at top to prevent heel contact</strong></li>
</ul>
```

The starred item will be bolded to match its emphasis in the notes.

#### 2. Add new "Short Game" accordion section (`index.html`, after Swing Thoughts)

Insert a new accordion item between "Swing Thoughts" and "Course Strategy":
```html
<div class="accordion-item">
  <button class="accordion-header" aria-expanded="false">
    <span>Short Game</span>
    <span class="accordion-icon">+</span>
  </button>
  <div class="accordion-body">
    <ul>
      <li>Pivot of body</li>
      <li>Throw w/ right arm to gauge distance</li>
      <li>Read putt from side; front first feel</li>
      <li>Speed in bunkers; use full swing</li>
      <li>Aimpoint from center of cup</li>
      <li>Accelerate putter like Bernhard</li>
    </ul>
  </div>
</div>
```

#### 3. (Optional) Add putter drill notes

The bottom of the page has notes about a ruler drill and planer slope system tied to Bryson's putting method. These could be added as either:
- A note appended to the existing **Vector Putting** section, or
- A new "Putting Drills" accordion item under Notes

This is optional — let me know if you'd like these included or saved for a separate task.

### Files Modified

| File | Change |
|------|--------|
| `index.html` | Replace swing thoughts list items (lines 82-87); insert new Short Game accordion item (after line 89) |

No CSS or JS changes needed — the new content uses the existing accordion pattern.