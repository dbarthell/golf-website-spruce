---
id: onboarding-create-artifact
type: task
title: 2. Create your first artifact
status: Todo
priority: High
---
# 2. Create your first artifact

Artifacts are how you track work in Spruce. Every feature, task, bug, chore, or memo is an artifact: a structured markdown document with frontmatter fields (status, priority, assignee, relationships) plus a freeform body. This task you're reading is one, too.

## Create one now

Find the **pencil icon** in the sidebar header at the top-left, just to the right of the project chip. Click it.

![The type picker dropdown showing bug, chore, feature, memo](resource://create-artifact.png)

A **type picker** drops down. Type to filter or click one of the five built-in types. For this walkthrough, pick **feature**.

The create modal opens with a title field, pinned-field chips along the bottom, and a **Create feature** button:

![The "New feature" create modal with title input, fields chips, and Create button](resource://create-artifact-modal.png)

- **Title** — required. Give it any name. "Test feature" is fine.
- **Fields chips** — click any chip (Fields / Status / Assignee / Priority) to set values before save. They're optional, and you can also set them later in the editor.
- **Keep open** toggle (bottom right) — leaves the modal open after Create, so you can fire off several artifacts in a row without re-clicking the pencil.
- **Create feature** — saves to disk.

Spruce writes a new markdown file to `artifacts/feature/SPR-xxxxx.md` inside your Spruce project folder, then drops you into the **artifact editor**.

## The artifact editor

![The artifact editor with field chips, body content, and tool panel on the right](resource://artifact-editor.png)

The editor has four regions:

**1. Breadcrumb bar (very top).** Shows the path "Kanban → Your title". The breadcrumb on the left takes you back to the view you came from. The `(...)` menu on the right has artifact-level operations (delete, duplicate, etc.).

**2. Tool panel tab strip (top right).** A row of tabs at the right end of the header: **Sessions**, **Git**, **Code**, **Comments**, **Activity**. Click any tab to open the **tool panel** on the right side of the editor. Click the active tab again to close it. The panel's open/closed state, active tab, and width are remembered per artifact, so context-switching back into an artifact drops you exactly where you left off.

Each tab is the artifact-scoped version of one of Spruce's global tool pages (the **TOOLS** section in the sidebar), with the same content but locked to this artifact:

- **Sessions** — terminal sessions running in this artifact's worktree. Run agent actions or plain shells from here. *(Available after **Start** is clicked, or always for plan-only projects with no linked code repo.)*
- **Git** — the artifact's git tool: branch info, smart action, view changes, commits, **Create PR**, and the red **Finish** button (which removes the worktree and deletes the branch when you're done). The tab label gains a `+N −N` badge when there are uncommitted changes. *(Available after **Start**.)*
- **Code** — file browser rooted at this artifact's worktree. *(Available after **Start**.)*
- **Comments** — every comment thread on this artifact (block-anchored on the body, top-level, or line-anchored in code), with reply and resolve controls. Use the panel for triage. Thread markers also render inline in the body when the panel is closed.
- **Activity** — recent edits, commits, comments, and field changes on this artifact, ordered by time.

You'll see only **Comments** and **Activity** for a brand-new artifact like the one you just created. The other three light up after you click **Start** (covered in step 4), which gives the artifact a branch and a worktree.

**3. Field chips (under the title).** The pinned fields for this type, rendered as inline chips:

![Close-up of field chips: type, Fields, Assignee, Priority, Status](resource://fields-chips.png)

- **Type chip** (leftmost) — change the artifact's template (e.g. feature → task).
- **Fields chip** — opens a panel with every field defined on the template, not just the pinned ones.
- **Status / Priority / Assignee / Size** chips — click to edit. Values come from the template's schema.

The chip text and colors come from the template you're using. Open **Settings → Templates** to add, remove, or rename fields.

**4. Body editor.** Below the chips. Full markdown editor: type, paste, format with the usual shortcuts (`Cmd+B`, `Cmd+I`, etc.). There's no save button. Spruce writes to disk as you type. You can use:

- **Lists, headings, code blocks, tables, links** — anything Spruce renders.
- **Block comments** — hover any paragraph, heading, or list item and a **comment icon** appears in the left gutter. Click it to start a thread anchored to that block (you'll meet comments properly in step 4).
- **Inline links to other artifacts** — type `@` followed by a title or ID, and Spruce inserts a clickable cross-link.
- **Drag-and-drop attachments** — drop any image, video, or file onto the body to attach it. Use the **paperclip icon** in the toolbar for picking from disk.

## Linking artifacts together

Features can have child **tasks** and **bugs**, and tasks and bugs can point back at their parent feature. Open the **Fields** chip and look for the **Tasks** / **Bugs** / **Feature** relationship fields, depending on the type. Click one and pick another artifact to link.

## Where it lives on disk

The artifact is a plain markdown file in your Spruce project folder: `artifacts/<type>/SPR-xxxxx.md`. Frontmatter holds the fields, the body is what you typed. You can `cat`, `grep`, or even hand-edit it in another editor. Spruce watches the folder and reloads when files change.

When you next click the **Sync** button in the sidebar header (step 5), this artifact gets committed and pushed alongside everything else in the project.

---

## Tour

- [1. Welcome to Spruce](artifact://onboarding-welcome)
- [2. Create your first artifact](artifact://onboarding-create-artifact) (you are here)
- [3. Find your work with views](artifact://onboarding-plan-page)
- [4. Hand work to an agent](artifact://onboarding-claude)
- [5. Sync your changes](artifact://onboarding-publish)
