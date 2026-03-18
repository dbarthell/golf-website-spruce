---
id: GOL-gi7k3m
type: feature
title: Golf Intelligence Green Map Integration
priority: Medium
size: XL
status: Backlog
---
# Golf Intelligence Green Map Integration

Integrate the Golf Intelligence API to show golfers a visual green map (slope + elevation images) and, where structured data is available, automatically derive a simplified planar slope input from real contour data.

---

## User Flow

1. Golfer selects their course and hole number (new course/hole selector UI)
2. **Green tab** — app displays the Golf Intelligence slope image (and optionally elevation image)
3. Golfer visually reads the break and fall line
4. **Calc tab** — golfer enters slope %, uses clock face to find aim point and backstroke length
5. Slope input is either manual (image-only path) or auto-derived (structured data path, if available)

---

## UI Layout

Two-tab structure within the existing dark hero section:

```
┌─────────────────────────────────────────────┐
│  [Course: Augusta — Hole 12  ▾]             │
├─────────────────────────────────────────────┤
│  [ Green ]  [ Calc ]                        │
├─────────────────────────────────────────────┤
│  (tab content)                              │
└─────────────────────────────────────────────┘
```

- **Green tab**: Slope image + elevation image (swipeable or toggled). Empty state if no course selected.
- **Calc tab**: Existing calculator UI — unchanged (stimp, distance, slope input, clock face, result panel)
- Course/hole selector sits above the tabs and applies to both views

---

## API Endpoints Used

| Endpoint | Cost | Purpose |
| --- | --- | --- |
| `/courses/searchCourseGroups` | 0 credits | Course search |
| `/greens/getSlopeImage` | 1 credit/green | Visual slope map |
| `/greens/getElevationImage` | 1 credit/green | Uphill/downhill context |

**Caching**: Store images in `localStorage` or `IndexedDB` keyed by `courseId + hole`. Valid for 1 year per user per the Golf Intelligence license terms.

---

## Slope Derivation (if structured data endpoint exists)

If Golf Intelligence provides slope vectors by GPS coordinate, auto-derive a single planar slope % using the following heuristic:

### Algorithm

1. **Sample** slope values along the putt line (ball position → hole)
2. **Discard the first third** — ball speed is high, slope has minimal influence on break
3. **Weight the middle third** — this is where the putt arc is shaped; use this for break direction and magnitude
4. **Use the final third / cup-end slope** — ball is dying here, most susceptible to break; use this to compute the ZBL position
5. **Net the slope vectors** along the weighted path (e.g. 2% east + 1% west = 1% east net)
6. Output a single slope % and direction → feed directly into existing calculator

### Why this works

- A single planar slope is already the model ZeroBreak uses — this just makes the input more accurate
- Middle-third weighting reflects putting physics: early ball speed dampens slope effect, late-stage slope determines break shape
- ZBL is anchored to cup-end slope because that's where the ball dies and is most deflected

### Fallback (image-only path)

If no structured data endpoint exists, the golfer reads the slope image visually and enters slope % manually — same as today, just with better visual context.

---

## Open Questions (needs answer from Chase)

1. **Does a structured slope data endpoint exist?** (slope % or vectors by GPS coordinate — not just image renders)
2. **Auth method** — API key in header? Bearer token?
3. **Green ID structure** — pass `courseId + hole number` directly, or is there a separate green ID?
4. **Predictive Putt Path** — timeline and response format (aim angle, clock face aim point, or both)? Relevant if we ever want to go beyond the planar slope model.

---

## Implementation Phases

### Phase 1 — Image-only (MVP)

- Course/hole selector UI
- Fetch + display slope image (1 credit/green, cached)
- Two-tab layout (Green / Calc)
- No change to existing calculator

### Phase 2 — Elevation

- Add elevation image to Green tab (swipeable alongside slope)
- Only if user feedback indicates uphill/downhill context is needed

### Phase 3 — Structured slope (if endpoint exists)

- Fetch slope vectors by coordinate along putt line
- Apply middle-third weighting heuristic
- Auto-populate slope input in Calc tab
- User can still override manually

---

## Related

- [GOL-15bq4q](artifact://GOL-15bq4q) — Pricing Model (API tier, subscription pricing, contract constraints)