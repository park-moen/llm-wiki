# Run agents in parallel

> Source: https://code.claude.com/docs/en/agents
> Collected: 2026-08-10
> Published: Unknown

Subagents, agent view, agent teams, and dynamic workflows each parallelize work in a different way. The right one depends on whether you want to stay in each conversation yourself, hand tasks off and check back later, or have Claude coordinate a group of workers for you.

Agent teams provide multiple coordinated sessions with a shared task list and inter-agent messaging, managed by a lead. They are experimental and disabled by default.

Worktrees give each session a separate git checkout, so parallel sessions never edit the same files. Use them for sessions you run yourself. Agent view moves each dispatched session into its own worktree automatically, and subagents you spawn can each get one too.

Running several sessions or subagents at once multiplies token usage.

The right approach depends on who coordinates the work, whether the workers need to communicate, and whether they edit the same files.

Do the tasks touch the same files? Isolate the work with worktrees. Subagents and sessions you run yourself can each use a separate worktree. Agent teams don’t isolate teammates in worktrees, so partition the work so each teammate owns a different set of files.
