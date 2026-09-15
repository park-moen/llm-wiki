# What is Orca?

> Source: https://www.onorca.dev/docs
> Collected: 2026-08-12
> Published: Unknown

A 60-second pitch: who Orca is for and when to reach for it.

Orca is a desktop IDE for running multiple AI coding agents side by side. Every task gets its own git worktree, its own agent terminal, and its own browser tab — so you can fan out work across Claude Code, Codex, Cursor CLI, and friends without stashing, branch-juggling, or losing flow.

![Orca main window: sidebar of worktrees, split panes with agent terminals and a diff view](https://www.onorca.dev/whats-new/posters/orca-split-screen.jpg) Orca main window: sidebar of worktrees, split panes with agent terminals and a diff view

## When to use Orca

- You want three agents trying the same bug in parallel and to pick the winner.
- You want to review AI-generated diffs seriously before you ship them.
- You already pay for Claude Code, Codex, or Cursor CLI and want one place to orchestrate them.
- You want agents to run remotely — over SSH, on a self-hosted Orca server, or in an on-demand VM — without giving up your IDE.

## Who it's for

Orca is designed for people who already write code for a living and want to use AI as leverage — not as a replacement. It assumes you read diffs, care about commits, and keep a worktree tidy. If you're looking for a no-code tool, Orca is not that.

## What Orca is not

- **Not a model.**Orca runs agents you already use — bring your own Claude, Codex, or OpenCode subscription.
- **Not a git replacement.**Every worktree is a real git worktree. You can `cd`in and use plain git whenever you want.
- **Not a hosted VPS product.**Orca runs on your desktop by default. Remote compute uses machines and cloud accounts you control — [SSH targets](https://www.onorca.dev/docs/ssh), [self-hosted Orca servers](https://www.onorca.dev/docs/remote-servers), or [Cloud VMs / per-workspace environments](https://www.onorca.dev/docs/ways-to-run#4-cloud-vms-per-workspace-environments).
Next steps

Head to [Install](https://www.onorca.dev/docs/install), then walk through [Your first 3-agent session](https://www.onorca.dev/docs/first-session) — the single most important page in these docs. When you're ready to move agents off the laptop, start with [Ways to run Orca](https://www.onorca.dev/docs/ways-to-run).

[Next → Install](https://www.onorca.dev/docs/install)

