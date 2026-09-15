# Run parallel sessions with worktrees

> Source: https://code.claude.com/docs/en/worktrees
> Collected: 2026-08-10
> Published: Unknown

A git worktree is a separate working directory with its own files and branch, sharing the same repository history and remote as your main checkout. Running each Claude Code session in its own worktree means edits in one session never touch files in another, so one session can build a feature while a second fixes a bug.

Worktrees are one of several ways to run Claude in parallel. They isolate file edits, while subagents and agent teams coordinate the work itself.

## Start Claude in a worktree

Pass `--worktree` or `-w` with a name to create an isolated worktree and start Claude in it. By default, the worktree is created under `.claude/worktrees/<name>/` at your repository root, on a new branch named `worktree-<name>`:

```bash
claude --worktree feature-auth
```

## Set up and clean up

A worktree is a fresh checkout, so initialize your development environment there. Gitignored files such as `.env` are not present unless copied separately; `.worktreeinclude` can copy selected gitignored files into new worktrees.

When a session exits, cleanup behavior depends on whether the worktree is clean and whether it contains changed files, untracked files, or new commits. Non-interactive runs do not clean up their worktrees automatically.

## Manage worktrees manually

```bash
git worktree add ../project-feature-a -b feature-a
git worktree add ../project-bugfix fix-issue-456
git worktree list
git worktree remove ../project-feature-a
```

Git commands in a worktree write to the main repository’s shared `.git` directory, so commands such as `git commit` work from inside a worktree.
