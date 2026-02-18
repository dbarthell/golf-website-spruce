---
id: GOL-328cks
type: feature
title: Simple website
status: In Progress
---
# Simple website

I want to make a simple golf website that shows my yardages (woods, irons, etc.) and my vector putting numbers I need to remember and possibly some other golf related info.

---

## Implementation Plan

### Overview

Build a clean, modern, single-page golf reference website. Since the goal is simplicity and the content is mostly static personal data (yardages, putting numbers), the site will be built with **plain HTML + CSS + vanilla JavaScript** — no framework overhead needed. The site will be responsive (phone-friendly for use on the course) and easy to update.

### Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Markup | HTML5 | Simple, no build step needed |
| Styling | CSS3 (with custom properties) | Modern, maintainable, no dependencies |
| Interactivity | Vanilla JS | Minimal needs — tab switching, possible expand/collapse |
| Dev Server | Live Server or `npx serve` | Quick local preview |
| Hosting | Static files (GitHub Pages, Netlify, or Vercel) | Free, simple deployment |

### Site Structure

```
golf-website/
├── index.html          # Single-page app shell
├── css/
│   └── styles.css      # All styles
├── js/
│   └── app.js          # Tab navigation & interactivity
├── data/
│   └── clubs.json      # Club yardage data (easy to update)
│   └── putting.json    # Vector putting reference data
├── images/             # Optional: logos, icons, course images
├── .gitignore
└── README.md
```

### Page Sections

The site will be a single page with a sticky top nav for quick section jumping:

#### 1. Header / Hero
- Site title (e.g., "My Golf Numbers")
- Clean, minimal branding with a golf-green color palette

#### 2. Club Yardages Section
- **Organized by category** using a tabbed or card-based layout:
  - **Woods** (Driver, 3W, 5W, etc.)
  - **Hybrids**
  - **Irons** (3i–PW)
  - **Wedges** (GW, SW, LW — with partial swing yardages if desired)
- Each club displayed as a card or table row showing:
  - Club name
  - Carry distance
  - Total distance (optional)
  - Notes (e.g., "stock shot", "punch", shot shape tendencies)
- Data loaded from `data/clubs.json` so yardages can be updated without touching HTML

#### 3. Vector Putting Numbers Section
- A reference card/table for vector putting values
- Organized by key metrics the user needs to recall on the course
  - Could include: stimp adjustments, slope %, break values, green speed references
- Clean, readable layout designed for quick glance on a phone

#### 4. Additional Golf Info Section (Expandable)
- A flexible area for other reference info, such as:
  - Pre-shot routine checklist
  - Course strategy notes
  - Swing thoughts / keys
  - Rules quick-reference
- Implemented as collapsible accordion cards so it stays tidy

### Design Approach

- **Mobile-first responsive design** — primary use case is checking numbers on the course from a phone
- **Dark/light mode** — dark header with a clean white/light-green content area
- **Golf-inspired palette**: greens (`#2d5016`, `#4a7c34`), off-white (`#f5f5f0`), subtle warm accents
- **Typography**: System font stack for speed; clean sans-serif headings
- **Sticky nav** at top for quick section jumps
- Large, readable text for yardage numbers (optimized for quick glances)

### Implementation Steps

- [x] **Step 1 — Scaffold the project**: Create `index.html`, `css/styles.css`, `js/app.js`, and `data/` directory structure
- [x] **Step 2 — Build the HTML shell**: Semantic layout with header, nav, and section containers for each content area
- [x] **Step 3 — Implement CSS styling**: Mobile-first responsive styles, color palette, typography, card/table layouts
- [x] **Step 4 — Create data files**: `clubs.json` with placeholder yardage data organized by club category; `putting.json` with vector putting structure
- [x] **Step 5 — Add JavaScript**: Load JSON data and render club cards/tables dynamically; implement tab switching and accordion behavior
- [ ] **Step 6 — Polish & test**: Responsive testing, accessibility pass, final visual tweaks
- [ ] **Step 7 — Deployment setup**: Add simple deployment instructions (GitHub Pages or similar)

### Data Format (clubs.json example)

```json
{
  "woods": [
    { "club": "Driver", "carry": 250, "total": 270, "notes": "Stock shot" },
    { "club": "3 Wood", "carry": 230, "total": 245, "notes": "" }
  ],
  "irons": [
    { "club": "7 Iron", "carry": 165, "total": 172, "notes": "Stock" }
  ],
  "wedges": [
    { "club": "PW", "carry": 140, "total": 145, "notes": "Full swing" },
    { "club": "PW", "carry": 120, "total": 125, "notes": "3/4 swing" }
  ]
}
```

### Open Questions for User

1. **What specific clubs do you carry?** (Driver, woods, hybrids, irons, wedges — full bag list with yardages)
2. **What are your vector putting numbers?** (What format/values do you track — stimp, break percentages, slope reads, etc.?)
3. **Any specific "other golf info" in mind?** (Swing thoughts, course notes, rules, etc.)
4. **Deployment preference?** (GitHub Pages, Netlify, Vercel, or just local?)
5. **Any branding preferences?** (Specific colors, a logo, your name/handle on the site?)