# Fix a UI bug with Design Mode

> Source: https://www.onorca.dev/docs/recipes/design-mode-fix
> Collected: 2026-08-12
> Published: Unknown

Design Mode collapses the "that button looks wrong" → "fixed commit" loop to under a minute.

## Steps

1. Open the worktree's browser pane. Navigate to the page with the bug.
1. Toggle [Design Mode](https://www.onorca.dev/docs/browser/design-mode) on.
1. Click the broken element. It lands in the agent chat as a rich attachment.
1. Type what you want fixed: "this padding is too tight, increase to match the cards above."
1. The agent edits the source. Hot reload refreshes the browser.
1. Click the element again to verify — if still wrong, repeat.
1. When it's right, commit.

## Why it's fast

No screenshot, no DOM hunting, no selector copying. The agent gets the HTML, computed CSS, and a cropped image of the exact element you pointed at — the same context a human reviewer would want.

[← Previous Jump between 10 worktrees](https://www.onorca.dev/docs/recipes/jump-worktrees)[Next → Work on a remote machine over SSH](https://www.onorca.dev/docs/recipes/remote-worktrees)

