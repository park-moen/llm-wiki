# git-worktree Documentation

> Source: https://git-scm.com/docs/git-worktree.html
> Collected: 2026-08-10
> Published: Unknown

## NAME

git-worktree - Manage multiple working trees

## DESCRIPTION

Manage multiple working trees attached to the same repository.

A git repository can support multiple working trees, allowing you to check out more than one branch at a time. With `git worktree add` a new working tree is associated with the repository, along with additional metadata that differentiates that working tree from others in the same repository. The working tree, along with this metadata, is called a "worktree".

This new worktree is called a "linked worktree" as opposed to the "main worktree" prepared by `git-init` or `git-clone`. A repository has one main worktree (if it’s not a bare repository) and zero or more linked worktrees. When you are done with a linked worktree, remove it with `git worktree remove`.

In its simplest form, `git worktree add <path>` automatically creates a new branch whose name is the final component of `<path>`. To instead work on an existing branch in a new worktree, use `git worktree add <path> <branch>`.

## Operational notes

Only clean worktrees (no untracked files and no modification in tracked files) can be removed. Unclean worktrees or ones with submodules can be removed with `--force`. The main worktree cannot be removed.

By default, `add` refuses to create a new worktree when `<commit-ish>` is a branch name and is already checked out by another worktree.

Each linked worktree has a private sub-directory in the repository’s `$GIT_DIR/worktrees` directory. Worktree-specific items such as `HEAD` resolve through the private directory, while shared refs resolve through `$GIT_COMMON_DIR`.

## BUGS

Multiple checkout in general is still experimental, and the support for submodules is incomplete. It is NOT recommended to make multiple checkouts of a superproject.
