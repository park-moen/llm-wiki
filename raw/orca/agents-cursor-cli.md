# Cursor CLI in Orca

> Source: https://www.onorca.dev/docs/agents/cursor-cli
> Collected: 2026-08-12
> Published: Unknown

Cursor CLI is Cursor's command-line agent. Orca runs it with first-class support — launch from the combobox, full OSC state detection, and restart chip on exit.

## Setup

1. Install Cursor CLI per [Cursor's docs](https://cursor.com/cli).
1. Log in once.
1. Orca auto-detects the CLI on `PATH`.

## Launching

Pick **Cursor**from the combobox. Orca launches the CLI scoped to the worktree. Cursor's TUI emits the state events Orca needs for agent state dots.

## Model selection

Model selection is driven by Cursor's own settings. Orca doesn't override it — configure inside the CLI.

[← Previous Codex in Orca](https://www.onorca.dev/docs/agents/codex)[Next → Hot-swap Codex accounts](https://www.onorca.dev/docs/agents/codex-hot-swap)

