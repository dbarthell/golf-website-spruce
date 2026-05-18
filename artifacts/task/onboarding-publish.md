---
id: onboarding-publish
type: task
title: 5. Sync your changes
status: Todo
priority: Low
---
# 5. Sync your changes

Your Spruce project is a git repository. Every time you create or edit an artifact, edit a template, save a new view, or change an action, the **pending-changes badge** ticks up. When you're ready to share with the team (or just want a safe checkpoint), you sync it.

Spruce keeps **two git workflows separate**:

- **The Spruce project itself** (artifacts, templates, views, actions, project config) — synced through the **sync menu in the sidebar header**.
- **Linked code repositories** (your actual source code) — synced through the **Git tab** inside an artifact's tool panel (the one you used in step 4).

This step is about the first one.

## The sync menu

Find the **orange chip in the sidebar header**, just below the project switcher. It shows a count of pending changes (e.g. *"3 pending"*) and a **Sync** button:

![The sync chip in the sidebar header showing pending changes and a Sync button](resource://sync-menu.png)

The badge updates live as you work. Right now it should already have a few items, since the Spruce project tracks your status changes on these onboarding tasks too.

Click anywhere on the chip to open the **sync menu**. Depending on whether the project has a remote and is up to date, you'll see:

- **Sync** — commits all pending changes and pushes to the remote in one step. Shown when the project has a remote.
- **Commit** — commits locally without pushing. Shown for projects without a remote (you can add one later from **Settings → Per-project settings**).
- **Pull** — appears when the local project is behind its remote.
- **Review changes** — opens the Activity page to see exactly what's about to be committed.

### What gets committed

The sync menu commits everything inside the Spruce project folder that's been touched since the last commit:

- New / edited / deleted artifacts
- Template changes (new fields, renamed types, etc.)
- View changes (renamed / new / deleted)
- Action edits
- Project config tweaks (focus bar filter, base branch, etc.)

It does **not** touch your linked code repos. Code changes need a commit on the code repo's own branch, from the Git tab inside the relevant artifact (or your usual terminal `git` workflow).

## Review before committing

The **Activity page** (sidebar → **TOOLS → Activity**) is a unified feed of:

- New / edited artifacts since the last sync
- Edited fields, with before/after values
- Modified templates / views / actions
- Recent commits on both the Spruce project and any linked code repos
- Uncommitted code changes across linked repos

![The Activity page showing pending Spruce and code changes side by side](resource://activity-panel.png)

Pick **Review changes** in the sync menu (or `⌘⌥3` to navigate there directly) to see exactly what's about to be committed before you hit Sync. Each row links back to the artifact / template it belongs to, so you can jump in and tweak if something looks off.

## Pulling teammates' work

There's no "pull" button you have to remember. Spruce checks for new commits on the project's remote when you open the project and on a quiet interval after that, so your teammates' changes show up on their own. When new commits arrive, you'll see them in the Activity feed.

If the local project is behind the remote, **Pull** appears in the sync menu so you can grab the latest before pushing.

## You're done with the tour

That's the everyday loop in Spruce: create an artifact, drive it through Plan / Implement / Review with an agent, push the code branch, sync the project.

A few things worth exploring next, all in **Settings** (sidebar bottom → **Settings**):

- **Templates** — add new artifact types or change which fields exist on the built-in ones.
- **Actions** — edit the four bundled actions, add your own (e.g. *Run dev server*, *Run tests*), import an action pack.
- **Agents** — pick the default agent per action, configure MCP for agents that need it (Kiro, OpenCode).
- **Per-project settings** — focus bar filter, base branch, linked code repositories.

And read the full docs at [buildwithspruce.com](https://buildwithspruce.com) for deeper coverage of comments, worktrees, the focus bar, keyboard shortcuts, and the rest.

## Clean up when you're ready

Once you've worked through the tour, get rid of these onboarding tasks:

1. Open each one, click the `(...)` menu in the artifact header, and pick **Delete**, or mark them **Done** so they fall out of active views.
2. (Faster) bulk-select them from the Kanban or Active work view: shift-click to range-select, then use the selection toolbar's **Delete** button.

You can also disable the tutorial seed for future projects: the wizard's *Project* step has an **Include tutorial artifacts** checkbox, and your preference is remembered for next time.

Happy building.

---

## Tour

- [1. Welcome to Spruce](artifact://onboarding-welcome)
- [2. Create your first artifact](artifact://onboarding-create-artifact)
- [3. Find your work with views](artifact://onboarding-plan-page)
- [4. Hand work to an agent](artifact://onboarding-claude)
- [5. Sync your changes](artifact://onboarding-publish) (you are here)
