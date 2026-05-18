---
id: onboarding-welcome
type: task
title: 1. Welcome to Spruce
status: Todo
priority: Urgent
---
# 1. Welcome to Spruce

You've just opened your first Spruce project. This is a short tour: five tasks that walk you through the things you'll do every day. Work through them in order, and mark each one **Done** (or delete it) as you go.

![A Spruce project showing features, tasks, and memos organized by status](resource://plan-page-overview.png)

## What Spruce is

Spruce is a native desktop app that combines **planning**, a **code browser**, **terminals**, and **AI coding agents** into one place. No browser tab, no cloud, no separate SaaS for tracking work.

The model has two roots that stay separate on disk:

- **The Spruce project** — a versioned folder holding your plan: artifacts, templates, views, and actions. Push it to a git remote and your whole team sees the same state after a pull.
- **Code repositories** — where your source code lives. Link zero, one, or many code repos to a Spruce project. The same plan can span multiple repos, or none at all if you're just tracking work.

Both are git repositories with separate remotes. The app weaves them together, but they sync independently.

## A quick tour of the app

Spruce is laid out as three regions:

### The sidebar

The left strip is your primary navigation. The numbered callouts in the screenshot below match the seven sections described after it. Read top to bottom.

![The full Spruce sidebar with numbered callouts for each section](resource://sidebar.png)

1. **Project switcher.** The chip showing the project's alias and color. Click it to switch between projects, or to open the wizard for a new one (**New Project…** / **Import Project…** at the bottom of the dropdown).
2. **New artifact** (pencil icon). Opens the type picker so you can create a feature, task, bug, chore, or memo without leaving the current view. You'll use this in step 2.
3. **Sync menu** (the orange *N pending* / **Sync** chip). Counts how many Spruce-project changes are waiting to be committed and pushed. Every artifact edit, template tweak, and view rename adds to this badge. Click it to commit and push the project (covered in step 5).
4. **Custom views.** The **VIEWS** section lists saved view configurations. The project ships with **Active work**, **Kanban**, and **Prioritization**. You're looking at Kanban right now. The `+` icon next to the section header opens a blank canvas you can shape and save as a new view.
5. **By Type views.** The **BY TYPE** section adds one row per artifact template (Bug, Chore, Feature, Memo, Task). Each row is a built-in view locked to that template. Click any to see only artifacts of that type. Filter / sort tweaks here persist locally in your browser (per project + template), not on disk.
6. **Global tools.** The **TOOLS** section holds the four cross-project pages: **Sessions** (running terminals), **Comments** (every comment thread), **Activity** (commits and pending changes), and **Code** (browse files in linked repos). When an artifact is open, the same hotkeys instead toggle the matching tab in the artifact's tool panel.
7. **Settings.** Global + per-project preferences: templates, actions, agents, focus bar filter, keyboard shortcuts, integrations, and more.

### The focus bar

A horizontal row of artifact chips pinned to the very top of the app, above the breadcrumb bar. It surfaces what's most relevant right now: artifacts assigned to you with active statuses, artifacts linked to your current git branch, anything with a running terminal session. It's the same on every page (views, by-type pages, artifact editor, settings) so your in-flight work is always one click away.

Click a chip to jump to that artifact. Hover it to pop up a menu of common actions (Start working, View changes, Commit, Push, Create PR, jump to a running session, Finish) without opening it.

### The canvas

The main work area, currently showing the Kanban view of your project's artifacts. Open any artifact (this one, for instance) and the canvas becomes the **artifact editor**, with a tool panel on the right for Sessions / Git / Code / Comments / Activity. You'll meet both surfaces in the next steps.

## Everything is a file

Your plan is plain markdown end-to-end. Artifacts (features, tasks, bugs, chores, memos), templates, saved views, and actions are all files in your Spruce project folder. You can `grep` them, `git diff` them, or open them in another editor. When you push the project, your teammates see the same state after a pull. Git *is* the sync layer.

## These tasks are real artifacts

The five tour items in this list (including the one you're reading now) are actual Spruce artifacts of type **task**, stored as plain markdown files in the project. You can edit them, change their fields, delete them, or mark them **Done** like any other artifact. They're not special. They ship in every new project (the wizard's *Include tutorial artifacts* checkbox controls whether they appear).

---

## Tour

- [1. Welcome to Spruce](artifact://onboarding-welcome) (you are here)
- [2. Create your first artifact](artifact://onboarding-create-artifact)
- [3. Find your work with views](artifact://onboarding-plan-page)
- [4. Hand work to an agent](artifact://onboarding-claude)
- [5. Sync your changes](artifact://onboarding-publish)
