---
id: onboarding-claude
type: task
title: 4. Hand work to an agent
status: Todo
priority: Medium
---
# 4. Hand work to an agent

Spruce treats AI coding agents as first-class collaborators. To put one on a piece of work, you **Start** the artifact. Spruce creates a branch and worktree, opens a Sessions panel, and lets you launch an agent into that worktree with the artifact's content as context.

The whole loop happens inside the artifact editor. You don't have to leave the artifact to ship code.

## Start the artifact

Open any feature or task (the one you created in step 2 works) and look at the **top-right of the artifact header**. The prominent filled **Start** button is there when the artifact has no branch yet.

![The artifact header with the Start button highlighted](resource://start-working.png)

Click it. Spruce will:

1. Create a git branch named after the artifact ID (e.g. `artifact/SPR-xxxxx`) in your linked code repo.
2. Create an isolated git **worktree** for that branch at `<repo>/.worktrees/SPR-xxxxx/`.
3. Open the **Sessions** tool panel on the right side of the editor, scoped to that worktree.

The Start button now changes shape. It becomes the **Finish** button (red, top-right). Click Finish later to remove the worktree and delete the branch when you're done with the artifact.

Each artifact gets its own worktree, so you can have several in-flight at once without branches stomping on each other.

## Open the Sessions panel

If the Sessions tool panel didn't auto-open, click **Sessions** in the tool panel tab strip (top right of the editor, the row of tabs including Sessions / Git / Code / Comments / Activity / Finish).

You can use the worktree directly: click **New Terminal** to drop into a regular shell at the worktree root and write code yourself. But the more interesting option is to launch an agent.

## Run an agent action

In the Sessions panel, click the **+** (or **New Terminal** if no sessions exist yet) and the **action picker** opens. New projects ship with four agent actions out of the box:

- **General** — a do-anything agent, opened in the worktree with the artifact as context. Good for ad-hoc work.
- **Plan** — the agent reads the artifact and proposes an implementation plan as comments on the artifact. Reply to its comments to refine.
- **Implement** — the agent writes code against the agreed plan.
- **Review** — the agent self-reviews the diff and flags issues.

![The Sessions panel showing the action picker with available actions](resource://actions-menu.png)

*(The exact list in the screenshot is from a project with custom actions added. Yours will start with **General**, **Plan**, **Implement**, **Review**, plus **New Terminal** and **New terminal (native)** for plain shells.)*

Pick an action (say **Plan**) and Spruce runs it in a new session. The agent reads the artifact's spec, drops you into the session UI, and starts writing.

### Switching the agent mode

Each action ships with **every supported agent mode pre-configured** in its `modes:` map: Claude Code, Codex, OpenCode, Kiro, Gemini. The first mode in the file is the default for that action. To switch:

- Open **Settings → Actions**, click an action (`plan`, `implement`, etc.), and use the **Set as active** menu to pick a different mode for this project.
- The choice is **per-action, per-project**, so you can drive `plan` with Claude and `implement` with Codex on the same project. Spruce stores the pick in the project's `active_modes` table.

## Review the agent's work

You review through two surfaces, both inside the artifact's tool panel.

### Comments on the artifact

The agent leaves block-anchored comments on the artifact body and top-level threads at the artifact level. They render **inline** with a gutter marker next to the block they're anchored to:

![An artifact with a comment thread anchored to a code block](resource://comments.png)

- Each comment shows the author: your name on user comments, the agent's runtime name (`claude` / `codex` / `kiro` / etc.) on agent comments.
- **Reply** to a comment to guide the agent. It picks up your reply on its next iteration, and if the action is configured for **comment forwarding**, even mid-session.
- Open the **Comments** tab in the tool panel for a panel listing every open thread on this artifact. Resolved threads disappear from inline but stay in the panel under "Show resolved".
- `⌘⇧/` toggles whether resolved threads are shown.

Comments are stored locally in `comments/` (a SQLite database in your Spruce project folder) and don't sync via git. They're the **in-flight loop** with your agent, not cross-team review.

### Diffs in the Git tab

Open the **Git** tab in the tool panel for everything related to the worktree's git state:

![The Git tab showing uncommitted diffs and the file tree](resource://git-tab.png)

The Git tab has a vertical strip of sections, top to bottom:

- **Actions** at the top: **Create PR**, **Update from main**, **Merge to main**, **Stash**.
- **Pull request** — the linked PR if there is one, with a link to open it on your git host.
- **View changes**: **All changes**, **Uncommitted**, **Branch commits**. Click any to drill into a diff view.
- **Commits** — the branch's commit history, one row each with stats and a click-to-view.

Inside any diff, you can leave **line-anchored code comments**. Hover the gutter and click the comment icon:

![Comments anchored to specific lines in a code diff](resource://agent-comments.png)

Use code comments for surgical feedback ("this should use the existing helper" / "extract this") without losing the context of the change. The agent reads these comments through Spruce's MCP tools the same way it reads body comments.

## Push and open a PR

When you're happy with the work, the **Git** tab is also where you ship it:

1. **Commit** — pick the file rows you want, write a message, click **Commit**.
2. **Push branch** — pushes the code branch to its remote.
3. **Create PR** — opens GitHub (or your git host) with the branch pre-selected for a new pull request.

These all happen on the **code repo**, not the Spruce project. The Spruce project itself (your artifacts, comments, etc.) has a separate sync flow that you'll meet in the next step.

---

## Tour

- [1. Welcome to Spruce](artifact://onboarding-welcome)
- [2. Create your first artifact](artifact://onboarding-create-artifact)
- [3. Find your work with views](artifact://onboarding-plan-page)
- [4. Hand work to an agent](artifact://onboarding-claude) (you are here)
- [5. Sync your changes](artifact://onboarding-publish)
