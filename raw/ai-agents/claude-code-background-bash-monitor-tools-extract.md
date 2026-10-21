# Tools reference — Background commands and Monitor (excerpt)

> Source: https://code.claude.com/docs/en/tools-reference
> Collected: 2026-10-17
> Published: Unknown

| `TaskOutput` | Removed in v2.1.277. Previously retrieved output from a background task; Claude reads a background task's [output file](/docs/en/interactive-mode#how-backgrounding-works) with `Read` instead | No |

| `TaskStop` | Stops a running background task by ID. It also accepts an [agent-team teammate](/docs/en/agent-teams) or a named background agent by agent ID or name. When no task matches the ID, the error lists the running background agents by ID and description, including agents that another agent spawned | No |

### Background commands

For long-running processes such as dev servers or watch builds, Claude can set `run_in_background: true` to start the command as a background task and continue working while it runs. List and stop background tasks with `/tasks`. After you stop one there, or from a connected client such as the desktop app, Claude moves on instead of waiting for it. If a subagent started the command, it's that subagent that moves on.

#### When a background command stops

A command that a [foreground subagent](/docs/en/sub-agents#run-subagents-in-foreground-or-background) started stops when that subagent's run ends, whether it finished, failed, or was interrupted. A command that the main conversation or a background subagent started keeps running after a final response, until it exits, is stopped, or reaches its [time limit](#time-limit-for-background-commands).

While a command that the main conversation started is still running, a run in non-interactive mode with the `-p` flag [stays open after its result](/docs/en/headless#background-tasks-at-exit) until that command exits or reaches its time limit. A command that a background subagent started is stopped when the run exits.

#### Time limit for background commands

In a session that runs unattended, such as a run with the `-p` flag, an Agent SDK application, a CI job, or a cloud session, background Bash and PowerShell commands have a time limit. A local session you work in from a terminal, the desktop app, or the VS Code extension has no time limit on background commands.

The time limit requires Claude Code v2.1.285 or later. Before v2.1.288, it applied in every session.

The time limit counts from the moment the command enters the background:

* A command that Claude starts in the background gets 30 minutes, or the `timeout` Claude passes with `run_in_background`, up to a maximum of 2 hours
* A command that starts in the foreground and then moves to the background, for example at its timeout, gets 30 minutes from the move

In a run with the `-p` flag whose prompt you pass as text rather than with `--input-format stream-json`, both defaults are 10 minutes instead of 30, because the run [waits for background commands after its result](/docs/en/headless#background-tasks-at-exit).

When a background command reaches its time limit, Claude Code stops it and tells Claude why, and Claude can start the command again with a longer `timeout` if the work still needs it. The stop notice reads `Background command "<description>" was stopped after reaching its background time limit`.

#### Raise the time limit for background commands

Two [environment variables](/docs/en/env-vars) raise these limits, for Bash and PowerShell commands alike. Both take milliseconds, and neither can shorten a limit: a lower value leaves the defaults and the 2-hour maximum in place.

* Set `BASH_DEFAULT_TIMEOUT_MS` above `1800000` to replace the 30-minute default with that value, both for commands Claude starts without a `timeout` and for moved commands. In a `-p` run whose prompt you pass as text, any value above `600000` replaces its 10-minute default
* Set `BASH_MAX_TIMEOUT_MS` above `7200000` to raise the 2-hour maximum to that value. Setting `BASH_DEFAULT_TIMEOUT_MS` above `7200000` raises the maximum the same way

#### Foreground commands that move to the background

When a foreground command reaches its timeout without finishing, Claude Code moves it to the background instead of stopping it, unless the command starts with `sleep`. A moved command's [time limit](#time-limit-for-background-commands) counts from the move, and a foreground subagent's moved command still stops when that subagent's run ends.

Setting [`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`](/docs/en/env-vars#variables) or running in [bare mode](/docs/en/headless#start-faster-with-bare-mode) disables auto-backgrounding along with the rest of the background task functionality, so a command that reaches its timeout stops instead.

The result of a command moved to the background states what happened:

* When the timeout triggers the move, the result reports it explicitly: `Command did not complete within its 120s timeout and was moved to the background`, with the seconds matching the timeout that applied, followed by the task ID and the path of the file the output is being written to.
* A `cd`, `pushd`, `popd`, or `chdir` inside a command that is moved to the background never carries over: the result states `Session cwd remains <dir>; directory changes made by the backgrounded command do not apply to subsequent commands.`, so Claude doesn't act on a directory change that didn't happen.

## Monitor tool

The Monitor tool lets Claude watch something in the background and react when it changes, without pausing the conversation. Ask Claude to:

* Tail a log file and flag errors as they appear
* Poll a PR or CI job and report when its status changes
* Watch a directory for file changes
* Track output from any long-running script you point it at
* Connect to a WebSocket feed and report each message as it arrives

For most watches, Claude writes a small script, runs it in the background, and receives each output line as it arrives. For a server that already pushes events, Claude can open a [WebSocket](#websocket-source) instead of running a script.

You keep working in the same session and Claude interjects when an event arrives.

Every watch Claude starts has a deadline: 5 minutes by default, at most 30 minutes, and at most 10 minutes in a [non-interactive](/docs/en/headless) run given a single prompt with `-p`.

At the deadline the watch ends. Claude gets one notice, so it can start the watch again if it's still needed.

Stop a monitor by asking Claude to cancel it or by ending the session. When you stop a [subagent](/docs/en/sub-agents) that started monitors, for example from `/tasks`, those monitors stop with it.

When Monitor runs a command, it uses the same [permission rules as Bash](/docs/en/permissions#tool-specific-permission-rules), so `allow` and `deny` patterns you have set for Bash apply here too. While [auto mode](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) is active, Claude Code sets aside allow rules that name `Monitor` itself, along with the other [broad allow rules it drops](/docs/en/permission-modes#how-the-classifier-evaluates-actions), so the classifier reviews Monitor commands the same way it reviews Bash commands.

The [WebSocket source](#websocket-source) has its own approval prompt, which the classifier also decides in auto mode.

The tool is not available on Amazon Bedrock, Google Cloud's Agent Platform, or Microsoft Foundry. It is also not available when `DISABLE_TELEMETRY` or `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` is set. On Windows, the tool is available only when [Git Bash](/docs/en/setup#set-up-on-windows) is installed.

Plugins can declare monitors that start automatically when the plugin is active, instead of asking Claude to start them. See [plugin monitors](/docs/en/plugins/components#monitors).

### WebSocket source

When a server already pushes events over a WebSocket, Claude can connect to it directly instead of writing a polling script. Each kind of socket activity either becomes an event or ends the watch:

* **Text messages**: each one becomes one event, even when the message spans multiple lines.
* **Binary messages**: not passed through. Claude receives a placeholder line such as `[binary frame, 512 bytes]` instead.
* **Messages larger than 1 MiB**: the watch ends, so subscribe to a filtered feed where one exists.
* **Socket close**: the watch ends and Claude receives the close code.

A WebSocket watch takes a `ws` input in place of `command`, and a single Monitor call can't combine the two. The `ws` input has two fields:

| Field | Required | Description |
| :- | :- | :- |
| `url` | Yes | The endpoint to connect to. Must be a `ws://` or `wss://` URL with no embedded credentials or whitespace, using ASCII characters only |
| `protocols` | No | WebSocket subprotocol names to offer during the handshake. Each entry must be a valid subprotocol token, and the list can't contain duplicates |

The `timeout_ms` deadline applies to a WebSocket watch too: the watch ends at the deadline, and `TaskStop` cancels it early.

Opening a WebSocket prompts for approval; in [auto mode](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) the classifier decides instead. The prompt doesn't offer an option to skip future prompts for the same host.

Claude Code denies URLs that point at a private, link-local, or cloud-metadata address, including hostnames that resolve to one. It also denies hosts in `sandbox.network.deniedDomains`, and when [`allowManagedDomainsOnly`](/docs/en/settings-reference#sandbox-network-allowmanageddomainsonly) is set in managed settings, any host outside the managed allowlist.
