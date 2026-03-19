---
id: GOL-ytdss2
type: feature
title: UI polish — logo size, onboarding copy, and chip colors
status: Done
---
# UI polish — logo size, onboarding copy, and chip colors

## Original requests
1. Make logo in main app a little bigger but keep header the same size
2. Incorporate "gravity doing the rest" into the second-to-last sentence of the first onboarding slide
3. Delete the Walk it off slide — find a different way in the app to communicate that tip
4. Restore the brown color for Calibrate selectors and Putt log day indicator instead of green

---

## Summary of changes

Four small but meaningful UI tweaks across the logo, onboarding copy, and chip/selector colors. Items 1, 2, and 4 are straightforward one-file edits. Item 3 is already done — see note below.

---

## Item 1 — Bigger logo, same header size

**File:** [`src/styles/global.css`](code://src/styles/global.css)

**Current state:** `.header-logo { height: 3rem; width: 3rem; border-radius: 8px; ... }` — already at 3rem × 3rem square.

**Change:** Increase to `3.5rem` or `3.75rem`. The header uses `align-items: center` flex layout, so the container height is driven by the logo. A bump to 3.5rem should remain within the existing header padding without visually enlarging the header bar; going to 3.75rem+ may push it slightly taller on 375px screens — worth a quick visual check.

**Suggested value:** `height: 3.5rem; width: 3.5rem` (safe increment) or `3.75rem` (more prominent).

> ⚠️ **Decision needed:** How much bigger — `3.5rem` or `3.75rem`? Can default to `3.5rem` and adjust.

---

## Item 2 — "Gravity does the rest" in onboarding slide 1

**File:** [`src/components/OnboardingModal.tsx`](code://src/components/OnboardingModal.tsx)

**Current second-to-last sentence (slide 1):**
> "Imagine a clock face where 6 o'clock is straight downhill; no matter your ball's position, you always aim at a point on that 12 o'clock line — the straight putt."

**The concept:** Gravity is what curves a putt that starts straight into the hole along the Zero Break Line — once you've aimed at the ZBL point, you putt straight and gravity handles the break.

**Proposed revision options (pick one):**

- *Option A:* "Imagine a clock face where 6 o'clock is straight downhill; no matter your ball's position, you always aim at a point on that 12 o'clock line — the straight putt — and let gravity do the rest."
- *Option B:* "Imagine a clock face where 6 o'clock is straight downhill; no matter your ball's position, you always aim at a point on that 12 o'clock line and putt straight — gravity curves the ball into the hole for you."
- *Option C (more evocative):* "Imagine a clock face where 6 o'clock is straight downhill; no matter where your ball sits, you always aim at one point on that 12 o'clock line — your straight putt — and let gravity do the curving."

> ⚠️ **Decision needed:** Which revision captures the right tone? Option A is the most minimal addition; Option B is the clearest mechanically; Option C adds flair.

---

## Item 3 — Delete the Walk it off slide

**File:** [`src/components/OnboardingModal.tsx`](code://src/components/OnboardingModal.tsx)

**Current state:** There are four slides: "The method," "Your backstroke," **"Walk it off"** (tag: "Measuring distance"), and "Make it yours." The Walk it off slide is slide 3 (index 2).

**Change:** Remove the Walk it off entry from the `SLIDES` array. The content is already covered by the inline dismissible `WalkOffTip` component on the main Lookup page (added in PR #63), which lives right below the distance/slope inputs — exactly where it's contextually useful. Removing the slide tightens the onboarding to three focused steps that flow directly into Calibrate.

**Note:** The `WalkOffTip` in `LookupPage.tsx` already serves as the "different way in the app" — no new component needed.

---

## Item 4 — Brown chip color for Calibrate selectors and Putt Log day indicator

**Files affected:**
- [`src/styles/global.css`](code://src/styles/global.css) — `.dist-chip-active`, `.log-round-chip-active`, and optionally the Stimp toggle light-variant indicator (`.stimp-sc-indicator-light`)

**Current state:** Both chips use `var(--green-dark)` = `#013c36` (deep forest green). The current palette already has a warm golden accent color: `--green-accent: #c9a86a`.

**The "brown":** The current palette's `--green-accent` (`#c9a86a`) is the warm sandy gold/bronze tone — almost certainly the color you're thinking of as "brown." Swapping the active chips from `--green-dark` (forest green) to `--green-accent` (golden brown) would give the warm, non-green look you want and also stays within the existing design system.

**Proposed change:** In `global.css`, update three selectors to use `--green-accent` instead of `--green-dark`:
1. `.dist-chip.dist-chip-active` — background + border-color
2. `.log-round-chip-active` — background + border-color
3. `.stimp-sc-indicator-light` — background (Stimp toggle on Calibrate page, for consistency)

Text color on the golden background would need to change from `var(--white)` to `var(--green-dark)` (dark forest green text on gold reads better than white on gold at this lightness level).

> ⚠️ **Decision needed:** Confirm `#c9a86a` (`--green-accent`) is the right brown. Also confirm: dark text or white text on the golden chip? Dark text (`--green-dark`) is likely more readable.

---

## Implementation sequence

1. **Item 4 (brown color)** — Confirm the brown hex, add a `--chip-brown` CSS variable, and apply to the three active-state selectors. Single file: `global.css`. (~5 min)
2. **Item 1 (bigger logo)** — One-line change in `global.css`. (~2 min)
3. **Item 2 (gravity copy)** — One-line string change in `OnboardingModal.tsx`. (~2 min)
4. **Item 3** — No changes needed; already done via the `WalkOffTip` component.

---

## Open questions before implementation

| # | Question | Blocking? |
|---|----------|-----------|
| 1 | Logo target height — `3.5rem` or `3.75rem`? (currently `3rem`) | No (can default to `3.5rem`) |
| 2 | Which gravity copy variant (A/B/C) do you prefer? | Yes — need copy approval |
| 3 | Confirm `--green-accent` (`#c9a86a`) as the brown — and dark text vs white text on it? | Yes |