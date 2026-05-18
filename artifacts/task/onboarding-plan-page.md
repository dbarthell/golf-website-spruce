---
id: onboarding-plan-page
type: task
title: 3. Find your work with views
status: Todo
priority: Medium
---
# 3. Find your work with views

There's no separate "all artifacts" page in Spruce. **Every list of work is a view**: a named configuration of filters, group-by, sort, and visible fields, applied to your project's artifacts. Pick a different one from the sidebar and the canvas reshapes itself.

![The artifact canvas: sidebar on the left, toolbar above, columns of artifacts below](resource://views.png)

## Views your project ships with

Look at the **VIEWS** section in the sidebar (left side). Three views are listed:

- **Active work** — features and tasks with status `Todo`, `In Progress`, or `In Review`, grouped by assignee. Use this when you want to see what's actually in flight.
- **Kanban** — every work item grouped by status. The one you're looking at right now. This is the project's default view.
- **Prioritization** — backlog items grouped by priority. Use this for grooming.

Click any view to switch to it. The active view is highlighted in the sidebar. Right-click a view (or click the `(...)` menu in the toolbar of the open view) for **Rename**, **Delete**, or **Set as default**.

The `+` button next to the **VIEWS** header creates a new search: a fresh canvas with no saved configuration. Save it from the toolbar's **Save** button once you've shaped it the way you want.

## The toolbar

The strip at the top of the canvas is where every view-shaping control lives. Read it left-to-right:

- **Type** (leftmost) — filter to a specific artifact template (feature / task / bug / etc.).
- **Filter** — add any field filter (status, priority, assignee, custom fields). Filters AND-combine. Remove a chip to widen.
- **Search** — text search across artifact titles.
- **Group by** (right side) — break the list into sections by any field. The current view groups by **status**.
- **Layout picker** — switch between board, list, and grouped layouts.
- **Fields** — toggle which fields show on each card.
- **Sort** — order within each group.
- **Save** / `(...)` — save changes back to the current view, save as a new view, or rename/delete.

![The toolbar with multiple filters applied and grouped columns below](resource://filter-group-sort.png)

Try it now: click **Filter** in the toolbar, pick **Priority**, and select **High**. The canvas narrows to just high-priority work. Click the chip again to remove it.

## The "By type" rows

Below **VIEWS** in the sidebar is a **BY TYPE** section, one row per artifact template (Bug, Chore, Feature, Memo, Task, and any custom types you add later). Each row is a built-in view locked to that template. Click one to see only that type.

The difference from saved views: filter / sort tweaks you make on a By type row persist **locally in your browser** (per project + template), not on disk. They don't travel through the Spruce project's git repo. Use a saved view when the configuration matters enough to share with the team; use a By type row when you want a quick personal lens on one type.

## The focus bar

The strip of artifact chips pinned to the very **top of the app** (above the canvas, above the breadcrumb) is the **focus bar**. It's the same on every page: views, by-type pages, the artifact editor, settings.

![The focus bar pinned above the active view with several artifact chips](resource://focus-bar.png)

What's in it:

- Artifacts **assigned to you** with status `Todo`, `In Progress`, or `In Review`.
- Artifacts **linked to your current git branch** (matches automatically when you create a branch named after an artifact ID).
- Artifacts with a **running terminal session**.

Click a chip to open that artifact. **Hover** a chip and a menu pops up with everything you can do without opening it: **Start working**, **View changes**, **Commit / Push / Create PR**, jump to a running **Sessions**, **Finish** (red, at the bottom). It's the fastest way to act on in-flight work.

To tune what shows up, open **Settings → Per-project settings → Focus bar filter**.

## Right-click and keyboard shortcuts

- **Right-click any artifact card** for a context menu of common ops (change status, change assignee, copy ID, delete, etc.).
- **Right-click a view** in the sidebar for rename / delete / set as default.
- `⌘K` opens the **Command Palette**. Type to jump to any artifact, view, or page.
- `⌘N` opens the **New artifact** type picker (same as the pencil icon).
- `⌘⌥1..5` jumps to the global tool pages (Sessions / Comments / Activity / Code).

See **Settings → Keyboard shortcuts** for the full list.

---

## Tour

- [1. Welcome to Spruce](artifact://onboarding-welcome)
- [2. Create your first artifact](artifact://onboarding-create-artifact)
- [3. Find your work with views](artifact://onboarding-plan-page) (you are here)
- [4. Hand work to an agent](artifact://onboarding-claude)
- [5. Sync your changes](artifact://onboarding-publish)
