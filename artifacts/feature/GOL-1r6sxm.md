---
id: GOL-1r6sxm
type: feature
title: Need a way to go back to onboarding slides
status: Done
---
# Need a way to go back to onboarding slides

Need a way to go back and read the onboarding slides from the main app if needed.

---

## Implementation Plan

### Summary

The onboarding slides are shown once to new users via `OnboardingModal.tsx`, rendered in `LookupPage.tsx` only when `localStorage.getItem('zerobreak-onboarded')` is falsy. Once a user completes or skips onboarding, the modal never appears again.

The fix is to add a **"How it works"** button to the main header that re-opens the onboarding slides in a "replay" mode. In replay mode the slides work the same way, except the final action ("Get Started") should return the user to the app rather than navigating to `/calibrate`, since they're already calibrated.

### Files to Change

| File | Change |
| --- | --- |
| [OnboardingModal.tsx](code://src/components/OnboardingModal.tsx) | Accept an `isReplay` prop; adjust end-of-slides button behavior |
| [LookupPage.tsx](code://src/pages/LookupPage.tsx) | Add `showOnboarding` state + "How it works" header button |

No new files, no new routes.

---

### Step-by-Step Changes

#### 1. `OnboardingModal.tsx` — add `isReplay` prop

Add an optional `isReplay?: boolean` to the `Props` interface.

- When `isReplay` is `true`:
- The final-slide button should say **"Done"** instead of **"Get Started"**
- Clicking it should call `onDismiss()` (close the modal) instead of navigating to `/calibrate?from=onboarding`
- The `dismiss()` function (called by "Skip for now") should **not** write to localStorage, since the user is already onboarded — just call `onDismiss()`
- When `isReplay` is `false` (default, first-time flow): no behavior change

```tsx
// Props change
interface Props {
  onDismiss: () => void;
  isReplay?: boolean;
}

// dismiss() — when replaying, skip the localStorage write
function dismiss() {
  if (!isReplay) localStorage.setItem('zerobreak-onboarded', '1');
  onDismiss();
}

// handleNext() — when replaying, "Get Started" just closes
function handleNext() {
  if (isLast) {
    if (!isReplay) localStorage.setItem('zerobreak-onboarded', '1');
    isReplay ? onDismiss() : navigate('/calibrate?from=onboarding');
  } else {
    setCurrent(c => c + 1);
  }
}

// Final button label
{isLast ? (isReplay ? 'Done' : 'Get Started') : 'Next'}
```

#### 2. `LookupPage.tsx` — add replay state + header button

Add a `showOnboarding` state (starts `false`) that forces the modal open even if the user is already onboarded.

```tsx
const [showOnboarding, setShowOnboarding] = useState(false);

// Render condition: show modal on first run OR when user requests replay
{(!onboardingDone || showOnboarding) && (
  <OnboardingModal
    isReplay={onboardingDone}
    onDismiss={() => {
      setOnboardingDone(true);
      setShowOnboarding(false);
    }}
  />
)}
```

Add a **"How it works"** button to the existing `header-links` div, alongside the Log and Calibrate buttons:

```tsx
<button
  className="full-view-link"
  onClick={() => setShowOnboarding(true)}
  aria-label="How it works"
>
  <IconInfoCircle size={20} stroke={2} />
  <span className="full-view-link-label">How it works</span>
</button>
```

`IconInfoCircle` is already available from `@tabler/icons-react` (same library as the other icons).

---

### Open Questions for Review

1. **Button placement** — should "How it works" go before or after the Log/Calibrate buttons? Currently proposing it goes *before* them (leftmost in the header-links group) since it's more of an informational action than a navigation action. Alternatively, it could go at the very right/end.
2. **Button label** — "How it works" fits the feature but is longer than "Log" or "Calibrate". Alternatives: "Guide", "Help", "Tutorial", or just the ℹ️ icon with no label. Should the label be shown on mobile or just the icon?
3. **"Skip for now" button in replay mode** — the skip button currently only appears on the last slide. In replay mode it arguably makes sense to show it on every slide (to close the modal early). Should we surface it on all slides when `isReplay` is true?
4. **"Get Started" → Calibrate in replay mode** — the current plan has the final button just close the modal. Should the replay version still offer a way to re-run calibration (e.g., offer "Go to Calibrate" as a secondary action on the last slide)?