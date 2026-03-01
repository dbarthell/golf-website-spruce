---
id: GOL-pm7r3k
type: task
title: Freemium model — free vs. paid feature split
status: Planned
---
# Freemium model — free vs. paid feature split

Define and implement a one-time unlock paywall that gates personalized/advanced features while keeping the core ZBL calculation and standard backswing reference free.

---

## Feature split

### Free (always available)

- ZBL aim calculation (distance + slope → aim)
- Standard backswing reference tables (uncalibrated, stimp-scaled)
- Clock position mode
- Stimp selection
- Onboarding

### Paid — one-time unlock

- **Personal backswing calibration** — adjust backswing distances to match the user's actual putting speed
- **Putt log + stats** — track outcomes per round; make %, miss tendency, putts per hole, 1/2/3-putt breakdown, miss scatter plot
- **Course profiles** — save stimp, calibration, and notes per course
- **ZBL calibration** *(future)* — personal break multiplier based on stroke type and logged outcomes

---

## Rationale

The backswing calculation without calibration remains free because:
- Gating it entirely makes the free experience feel broken (users can aim but not hit the right distance)
- Generic reference tables are available elsewhere — keeping them free retains users in the app

Calibration is the natural paid gate because:
- It requires user effort and delivers personalized value
- It's the "makes it yours" story already established in onboarding
- Users who calibrate are engaged and understand the method — the right audience to convert

A **one-time unlock** fits better than a subscription because:
- The app is a utility, not a service — no ongoing infrastructure cost justifies recurring billing
- Golf is seasonal; recurring charges during off-season drive cancellations
- Subscription tier only makes sense if/when cloud sync, a course database, or AI features are added

---

## Implementation notes

- Paywall gate should check a local `localStorage` flag (e.g. `zb-unlocked`) after purchase
- Trial period: full access for a defined number of rounds or days, then prompt to unlock
- Unlock prompt should appear naturally — e.g. after the trial limit is hit mid-calibration or when viewing the log
- No features should be hidden in the UI entirely during trial — show them, let the user engage, gate on save/persist
- Payment integration TBD (likely RevenueCat for iOS PWA or a simple Stripe one-time link)

---

## Open questions

- Trial length: number of rounds (e.g. 3) vs. time-based (e.g. 14 days)?
- Price point for one-time unlock
- Whether to offer a "restore purchase" flow for reinstalls
- App Store / PWA distribution affects payment options — clarify before building paywall logic
