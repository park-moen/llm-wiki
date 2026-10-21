# Developer commands — Background terminals (excerpt)

> Source: https://learn.chatgpt.com/docs/developer-commands?surface=cli
> Collected: 2026-10-17
> Published: Unknown

### Check background terminals with `/ps`

1. Type `/ps`.
2. Review the list of background terminals and their status.

Expected: Codex shows each background terminal's command plus up to three
recent, non-empty output lines so you can gauge progress at a glance.

Background terminals appear when `unified_exec` is in use; otherwise, the list may be empty.

### Stop background terminals with `/stop`

1. Type `/stop`.
2. Confirm if Codex asks before stopping the listed terminals.

Expected: Codex stops all background terminals for the current session. `/clean`
is still available as an alias for `/stop`.
