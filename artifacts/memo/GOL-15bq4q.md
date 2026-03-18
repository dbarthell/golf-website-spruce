---
id: GOL-15bq4q
type: memo
title: Pricing Model
---
# Pricing Strategy

## Summary

ZeroBreak's premium tier is powered by the **Golf Intelligence API** (Chase Stracka / golfintelligence.com). Based on the API Partnership Program and projected usage, here's the recommended pricing strategy.

---

## API Cost Structure

ZeroBreak only needs two endpoints:

| Endpoint | Cost |
|---|---|
| Course Search (`/courses/searchCourseGroups`) | 0 credits — free |
| Green Slope Image (`/greens/getSlopeImage`) | 1 credit / green |
| Green Elevation Image (`/greens/getElevationImage`) | 1 credit / green |

**Per-user math:**
- 2 credits/green × 18 greens = **36 credits/course**
- 3 courses loaded/year (Golf Intelligence's avg assumption) = **108 credits/user/year = ~9 credits/user/month**
- Courses are cached locally for **1 year per user** — no re-fetching

---

## API Tier Recommendation: Starter ($399/mo)

| Tier | Monthly Cost | Credits/Mo | Est. Premium Users Supported |
|---|---|---|---|
| **Starter** | $399 | 10,000 | ~1,100 users |
| Basic | $999 | 30,000 | ~3,300 users |
| Pro | $2,499 | 100,000 | ~11,100 users |

Start on **Starter** at $399/mo. At ~1,100 users, your API cost per user is **~$0.36/user/month** — well below any reasonable subscription price.

---

## ZeroBreak Subscription Pricing

### Recommended: **$4.99/mo** or **$49.99/year**

| Metric | Value |
|---|---|
| API cost per user/month | ~$0.36 |
| Recommended price | $4.99/mo |
| Break-even subscribers | **80 users** |
| Margin per user above break-even | ~$4.63/mo |

**Why $4.99/mo:**
- Impulse-buy threshold for golfers
- Comparable to premium Golf GPS apps
- API cost is a rounding error — margins are excellent
- Room to raise later once you have retention data

**Why offer $49.99/year alongside it:**
- 2 months free vs monthly ($59.88) — standard convention, not over-discounting
- Improves cash flow and reduces churn
- Industry norm for golf/sports apps

---

## Constraints to Keep in Mind (from Contract)

- **No derivatives**: Don't programmatically extract slope data from images to power your own calculations — the contract prohibits derivative use
- **Caching**: One course download is valid for 1 year per user; another user loading the same course requires new credits
- **No reselling**: Data cannot be resold outside the consumer app

---

## Implementation Plan

### Step 1 — Get Access
Chase sent the pricing doc but not an API key yet. Reply to him (or email chase@golfintelligence.com) to sign up for the **free 200-credit test tier**. That gets you credentials to experiment with before committing to Starter.

### Step 2 — Course + Hole Selection UI
Add a new flow to the LookupPage (or a separate page):
1. User types course name → hits `/courses/searchCourseGroups` (free)
2. App shows matching courses, user picks one
3. User picks hole number (1–18)
4. App calls `/greens/getSlopeImage` (1 credit) → displays the image

### Step 3 — Cache Aggressively
1 credit per green image, and the license allows **1-year local caching per user**. Store the image URL (or image itself) in `localStorage` or `IndexedDB` keyed by `courseId + hole`. User only pays the credit once per course per year.

### Step 4 — Wire into Existing Flow
The slope image sits alongside the existing `SlopeInput` + `ClockFace` — user visually reads the break from the image and enters it manually like they do now. No change to the math.

---

### Still Needed from Chase (blockers)
- Actual API base URL and auth method (API key in header? Bearer token?)
- Whether there's a hole-level green ID or if you pass `courseId + hole number` directly

The PDF is marketing, not a dev spec — you need the actual API docs to build against.

---

## Next Steps

1. Confirm with Chase whether a structured **slope data endpoint** (numeric values, not just image) exists — would unlock better math integration
2. Confirm the **Predictive Putt Path** timeline if you ever want to revisit that feature
3. Sign up for **Free Test tier (200 credits)** to validate the green image integration before committing to Starter

---

## Previous Conversation

❯ hmm or couldn't we use our own math based on inferring the slope from the image and the golfer's      

location?                                                                                               

⏺ Clever idea, but a few challenges:

1. Terms — the contract explicitly prohibits "derivatives." Programmatically extracting slope data

  from their images to power your own calculations could be interpreted as a derivative use. Worth

  clarifying with Chase before building it.

2. Accuracy — the slope image shows vectors/colors representing slope direction and magnitude, but

  you'd need to reverse-engineer their color scale to get meaningful % values. It'd be approximate at

  best.

3. GPS mapping — to know which pixel corresponds to the golfer's position, you'd need to map GPS

  coordinates to image coordinates. That's doable but requires the GPS endpoint (2 credits) plus knowing

   the image's geographic bounds.

  It's technically possible but fragile, potentially a terms violation, and a lot of engineering for

  approximate results.

  Honestly the cleaner path is to just ask Chase two things:

1. Is there a structured slope data endpoint (not just image) — many of these APIs have one that's not

   in the marketing doc

2. What's the timeline on Predictive Putt Path and what does it return exactly

  If they have raw slope data by coordinate, that's the integration you want. If not, the Putt Path

  endpoint is your premium feature and the slope image is just a nice visual to show alongside your

  existing calculator.

  Want me to draft a reply to Chase?

❯ yes please                                                                                            

⏺ Here's a draft:

  ---

  Hi Chase,

  Thanks for sending this over. A couple of questions before I dig in:

1. Is there a structured slope data endpoint (slope % or vector values by coordinate) in addition to

  the image renders? The image works for visualization, but my app calculates backswing and aim

  mathematically so I need numeric slope input.

2. What's the timeline on Predictive Putt Path, and what does the response look like — aim angle, aim

  point on clock face, both?

  Those two answers will determine how I integrate this.

  Dan

  ---

  Short enough? Want me to adjust the tone or add anything?

❯ why doesn't the copy basic text command work? i have it selected?                                     

⏺ That's outside my visibility — I can't see your screen or what's selected. Are you trying to copy text

   from the Claude Code terminal, or from somewhere else (the PDF, a browser, etc.)?

❯ yeah from the terminal                                                                                

⏺ Claude Code renders output in a terminal UI that doesn't always play nicely with standard text

  selection/copy. A few things to try:

- Hold Option while clicking and dragging to select text (bypasses terminal mouse handling)
- Right-click → Copy
- If you're in iTerm2, Cmd+Shift+C sometimes works when Cmd+C doesn't

  Which terminal are you using?

❯ actually i think we can make do with just the images                                                  

⏺ Yeah that makes sense — show the golfer the green slope image so they can visually read the break, and

   they still enter the slope % manually like they do now. The image just makes that estimate way more

  informed than eyeballing it on the course.

  Want to simplify the Chase reply to just ask about the Putt Path endpoint then?

❯ so what all would we use again? we don't need the predictive path feature                             

⏺ For ZeroBreak's premium tier you'd really only need:

- Course Search — free, let users find their course
- Green Slope Image — 1 credit per green, show the visual alongside the calculator
- Green Elevation Image — 1 credit per green, optional but nice to have

  That's 2 credits per green if you include elevation, or 1 credit if slope only. With caching (1 year

  per user), a user loading a full 18-hole course costs 18–36 credits one-time.

  Pretty lightweight. Starter tier at $399/mo would cover a lot of users.

❯ so how much to break even                                                                             

⏺ Depends on your premium subscription price. What are you thinking of charging per month?

❯ not sure                                                                                              

⏺ Let's work backwards. At Starter tier ($399/mo):

  Assuming a user loads 3 courses/year and you cache slope + elevation (2 credits/green × 18 greens = 36

   credits/course × 3 courses = 108 credits/user/year = 9 credits/user/month):

- 10,000 credits/month ÷ 9 = \~1,100 premium users supported
- Break even: $399 ÷ 1,100 = $0.36/user/month

  So basically any reasonable subscription price ($3, $5, $10/mo) covers the API cost easily. The

  $399/mo is your fixed cost until you scale past \~1,100 users.

  At $4.99/mo you'd break even at 80 subscribers. Everything above that is margin.

❯ is that what you'd recommend?                                                                         

⏺ For an early-stage putting app, $4.99/mo feels right. A few reasons:

- Low enough that it's an impulse buy for a golfer
- Comparable to what Golf GPS apps charge for premium tiers
- Your API cost per user is essentially $0.36 — margins are excellent
- Easy to raise later once you have users and feedback

  I'd consider a $39.99/year option alongside it — gives golfers a discount, improves your cash flow,

  and reduces churn. A lot of golf apps do monthly + annual.