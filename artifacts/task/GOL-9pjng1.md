---
id: GOL-9pjng1
type: task
title: Backswing reference table enhancements
status: Done
---
# Backswing reference table enhancements

Let’s add 9, 12, and 15 inches past outside to the table and make the numbers dependent on the stamp selection at the top like the rest of the page.

## Implementation (React app — src/)

**Backswing reference table enhancements** ([BackswingTable.tsx](code://src/components/BackswingTable.tsx), [LookupPage.tsx](code://src/pages/LookupPage.tsx), [backswing.ts](code://src/lib/backswing.ts)):

1. Exported `getDistanceForStimp` from `lib/backswing.ts`.
2. `BackswingTable` now accepts a `stimp` prop and uses `getDistanceForStimp(r, stimp)` for the Distance column instead of the hardcoded `r.stimp10`.
3. Filter expanded to include 9", 12", 15" past outside rows (24", 27", 30" backswing inches, `original: false`) via `EXTENDED_INCHES = [24, 27, 30]`.
4. Title now renders as "Backswing Reference (Stimp X)" using the live stimp value.
5. `LookupPage` passes `stimp={stimp}` to `BackswingTable` so it re-renders when stimp changes.

**Additional bugs fixed in this session:**

- **Onboarding layout (cut off / shifted left)** ([OnboardingModal.tsx](code://src/components/OnboardingModal.tsx)): Replaced Mantine `<Modal fullScreen>` with a plain `<div className="onboarding-overlay">` — the existing CSS for `.onboarding-overlay` handles full-screen centering correctly without Mantine interference. Button on last slide now dismisses onboarding and returns to the main page ("Get Started").
- **Result panel numbers unreadable** ([LookupResult.tsx](code://src/components/LookupResult.tsx)): Replaced Mantine `<Paper>` with a plain `<div className="lookup-result">` so the CSS `background: var(--white)` applies reliably without Mantine's CSS variable system overriding it.
- **CSS cleanup** ([global.css](code://src/styles/global.css)): Removed unused `.onboarding-modal-content` / `.onboarding-modal-body` and `.lookup-result.mantine-Paper-root` overrides.