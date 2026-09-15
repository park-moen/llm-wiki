# Per-worktree browser

> Source: https://www.onorca.dev/docs/browser/overview
> Collected: 2026-08-12
> Published: Unknown

Every Orca worktree has its own browser. It's a real Chromium window — address bar, history, devtools — embedded in a pane. Tabs are scoped to the worktree, so the app you're building against stays out of the way of your other work.

![Per-worktree browser pane with address bar and tab strip](https://www.onorca.dev/whats-new/posters/orca-design-mode.jpg) Per-worktree browser pane with address bar and tab strip

## Controls

- Address bar with history and fuzzy URL completion.
- Back / forward / reload / stop. Hover the reload control for the normal-reload shortcut; right-click (or long-press) for **Reload**and **Hard Reload**with their shortcuts — hard reload bypasses the cache when you're iterating on local frontend assets.
- `Cmd-F`— find in page.
- `Cmd-T`— new tab, scoped to this worktree.
- `Cmd-Shift-T`— reopen last closed tab.

## Worktree scoping

Tabs are filtered per worktree. Switching worktrees restores that worktree's browser tabs and scroll positions.

![Per-worktree browser tabs with persistent sessions — one click imports cookies from Chrome or Edge so you stay signed in.](https://www.onorca.dev/whats-new/posters/browser-2-demo.jpg) Per-worktree browser tabs with persistent sessions — one click imports cookies from Chrome or Edge so you stay signed in.

## Link routing

Under [Settings → Browser → Link Routing](https://www.onorca.dev/docs/settings), choose whether http(s) links from the terminal, markdown, and editor open in Orca's per-worktree browser or the system browser.

A nested **Hold Shift…**toggle inverts that default for one click with the platform modifier (`⇧⌘-click`on macOS, `Shift+Ctrl+click`on Windows / Linux):

- When links open in **Orca**, the modifier sends one link to the system browser.
- When links open in the **system browser**, turn the toggle on so the modifier opens one link in Orca's built-in browser instead (plain click stays system).

Remote/SSH sources still never open inside Orca's browser, even with the modifier.

## Downloads

Browser downloads appear in a shelf under the toolbar while they are active or recently completed, with actions to cancel an in-progress download, open a finished file, show it in its folder, or dismiss the row.

## Share as artifact

Local HTML files opened in the worktree browser can use **Share as artifact**in the toolbar to mint a public view link through your signed-in Orca account (same opt-in gate and management surface as Markdown). Relative HTML assets are not uploaded — share a self-contained file or use absolute asset URLs. See [Settings → Artifacts](https://www.onorca.dev/docs/settings#artifacts) and [CLI reference → Artifacts](https://www.onorca.dev/docs/cli/reference#artifacts).

## Viewport-size emulation

Set a custom viewport size on a browser tab to test responsive layouts without resizing the whole pane. Orca uses Chrome DevTools Protocol device emulation under the hood, so the page sees the emulated dimensions in `window.innerWidth`and media queries.

## Automation

The browser is also scriptable by agents via the [Orca CLI](https://www.onorca.dev/docs/cli/overview) — `orca snapshot`, `orca click`, `orca fill`, and so on. Same browser you interact with, same tabs.

![Agent driving Orca's built-in browser via the CLI — same tabs, same session](https://www.onorca.dev/whats-new/posters/orca-browser-use.jpg) Agent driving Orca's built-in browser via the CLI — same tabs, same session

## Next steps

- [Design Mode](https://www.onorca.dev/docs/browser/design-mode) — turn the browser into a pointer-to-code feedback loop.
- [Browser-use profiles](https://www.onorca.dev/docs/browser/profiles) — run the browser with a specific login, cookie jar, or user agent.
[← Previous File explorer & external drag-drop](https://www.onorca.dev/docs/editing/file-explorer)[Next → Design Mode](https://www.onorca.dev/docs/browser/design-mode)

