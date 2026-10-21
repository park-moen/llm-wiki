# Interactive mode — Background Bash commands (excerpt)

> Source: https://code.claude.com/docs/en/interactive-mode
> Collected: 2026-10-17
> Published: Unknown

## Background Bash commands

Claude Code supports running Bash commands in the background, allowing you to continue working while long-running processes execute.

### How backgrounding works

When Claude Code runs a command in the background, it runs the command asynchronously and immediately returns a background task ID. Claude Code can respond to new prompts while the command continues executing in the background.

To run commands in the background, you can either:

* Prompt Claude Code to run a command in the background
* Press `Ctrl+B` to move a regular Bash tool invocation to the background. Tmux users must press `Ctrl+B` twice due to tmux's prefix key.

When a command reaches its timeout before it finishes, Claude Code automatically [moves it to the background](/docs/en/tools-reference#foreground-commands-that-move-to-the-background) instead of stopping it, unless the command starts with `sleep`. If you've turned background tasks off with [`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`](/docs/en/env-vars#variables) or by starting in [bare mode](/docs/en/headless#start-faster-with-bare-mode), the command stops at its timeout. To change the timeout, set the [Bash timeout environment variables](/docs/en/tools-reference#timeout-and-output-limits).

**Key features:**

* Output is written to a file and Claude can retrieve it using the Read tool
* Background tasks have unique IDs for tracking and output retrieval
* Background tasks are automatically cleaned up when Claude Code exits. On macOS and Linux, when you stop a background task from [`/tasks`](/docs/en/commands) or Claude Code stops it at exit, processes that detached from the task's shell, such as ones started under `setsid` or `timeout`, stop too
* If you background the session instead of exiting it, your background tasks keep running in the background session. See [background a running session](/docs/en/agent-view#from-inside-a-session)
* Background tasks are automatically terminated if output exceeds 5GB, with a note in stderr explaining why
* On macOS and Linux, Claude Code stops your running background tasks when the operating system reports critical memory pressure, provided the session has been idle for at least 30 minutes and no turn or subagent is running
  * The [debug log](/docs/en/debug-your-config) says why tasks were stopped, or why a pressure event left them running
  * Set [`CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP`](/docs/en/env-vars) to `1` to turn off memory-pressure stops
* Background commands in a local session you work in from a terminal, the desktop app, or the VS Code extension have no time limit. In a session that runs unattended, such as a `-p` run or a cloud session, Claude Code stops a background command at its [time limit](/docs/en/tools-reference#time-limit-for-background-commands)
* A background command that a foreground [subagent](/docs/en/sub-agents#run-subagents-in-foreground-or-background) started ends when that subagent's run ends, whether it finished, failed, or was interrupted; see [When a background command stops](/docs/en/tools-reference#when-a-background-command-stops) in the tools reference

To disable all background task functionality, set the [`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`](/docs/en/env-vars#variables) environment variable to `1`. Starting in [bare mode](/docs/en/headless#start-faster-with-bare-mode) turns it off as well.

**Common backgrounded commands:**

* Build tools (webpack, vite, make)
* Package managers (npm, yarn, pnpm)
* Test runners (jest, pytest)
* Development servers
* Long-running processes (docker, terraform)

### Shell mode with `!` prefix

Run shell commands directly without going through Claude by prefixing your input with `!`:

```bash theme={null}
! npm test
! git status
! ls -la
```

Shell mode:

* Adds the command and its output to the conversation context
* Shows real-time progress and output
* Supports the same `Ctrl+B` backgrounding for long-running commands
* Doesn't require Claude to interpret or approve the command
* Supports history-based autocomplete: type a partial command and press `Tab` to complete from previous `!` commands in the current project
* Supports live file path autocomplete: type a token containing a forward slash, such as `./src/` or `~/`, to see a dropdown of matching files and directories, then press `Tab` to accept. Use forward slashes on Windows too; the dropdown is triggered by `/`, not `\`
* Exit with `Escape`, `Backspace`, or `Ctrl+U` on an empty prompt
* Pasting text that starts with `!` into an empty prompt enters shell mode automatically, matching typed `!` behavior

Unless your session is one of those listed under [strict sandbox mode](/docs/en/sandboxing#turn-off-the-retry-with-strict-sandbox-mode), commands you type in shell mode run outside the [sandbox](/docs/en/sandboxing) even when you've enabled sandboxing, because the sandbox applies to the commands Claude runs.

Claude responds to the command output automatically once it lands in the transcript, so you can run `! npm test` and get an explanation of the failures without a second prompt. The response costs the same as sending a normal prompt. To add the output to context without a response instead, set [`respondToBashCommands`](/docs/en/settings-reference#respondtobashcommands) to `false` in `settings.json`.
