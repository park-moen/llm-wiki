# Stagehand Codex integration (excerpt)

> Source: https://docs.stagehand.dev/v4/integrations/codex
> Collected: 2026-09-25
> Published: Unknown

Integrations
# Codex

Give a Codex agent persistent Stagehand browser tools over MCP/stdio.

Copy page Copy page

The Codex integration gives a Codex agent a persistent Stagehand browser it can drive with `run`, `snapshot`, and `screenshot`. A Codex SDK thread connects to the Stagehand facade MCP server over MCP/stdio, and one server process owns the browser, so page state survives across tool calls.

Stagehand ships this experimental integration from the repository rather than publishing it as a standalone adapter.
##

cite48†​ L102: 
Prerequisites

  * Node.js 24 or newer
  * pnpm 11.10.0
  * An OpenAI API key or an existing `codex login` for the example agent
  * A current Google Chrome installation for local browser mode
Quickstart

1

Clone and build Stagehand

    `git clone https://github.com/browserbase/stagehand.git
    cd stagehand
    pnpm install --frozen-lockfile
    pnpm exec turbo run build \
      --filter @browserbasehq/stagehand-integrations
    `

2

Configure the Codex agent

    `export OPENAI_API_KEY="your-openai-api-key"
    `

Codex picks its own harness-tuned default model. Set `CODEX_STAGEHAND_MODEL` to override it.

3

Choose the browser
The example defaults to Browserbase when `BROWSERBASE_API_KEY` is set, otherwise it uses local Chrome:

    `export STAGEHAND_BROWSER="browserbase"
    export BROWSERBASE_API_KEY="your-browserbase-api-key"
    `

4

Run a browser task

    `pnpm --dir packages/integrations/codex start -- \
      "Open https://example.com and report the page title."
    `
Configuration
Variable  | Purpose
--- | ---
`CODEX_STAGEHAND_MODEL`  | Optional model override; by default Codex uses its own harness-tuned model.
`OPENAI_API_KEY`  | Credential for the example agent. Codex does not forward it to the MCP child.
`CODEX_PATH_OVERRIDE`  | Optional path to a `codex` binary when the SDK’s vendored binary is missing.
`STAGEHAND_BROWSER`  | Select `local` or `browserbase`.
`BROWSERBASE_API_KEY`  | Required for Browserbase.
`BROWSERBASE_PROJECT_ID`  | Optional Browserbase project ID.
`STAGEHAND_MODEL_NAME`  | Optional model for Stagehand AI methods called inside `run`.
`STAGEHAND_MODEL_API_KEY`  | Required with `STAGEHAND_MODEL_NAME`; the MCP child does not receive agent-provider credentials.
##

cite51†​ L170: 
Keep the browser session alive
The example starts one Codex thread against one facade MCP server, with raised MCP startup and tool timeouts because browser launches exceed the Codex defaults. The local sandbox stays read-only; the browser work happens in the MCP server. Preserve that lifetime if you adapt the integration; a new process per tool call starts a new browser. Codex has no native turn limit; long tasks run until the model finishes.
The MCP child receives only Stagehand and Browserbase configuration plus the process values required to launch Node. The host’s model credential remains in the Codex process.
##

cite52†​ L177: 
Connect a running Codex CLI

The same server works from the Codex CLI. Merge the package’s `config.toml` into `~/.codex/config.toml`, adjusting the path to your checkout:

    `[mcp_servers.stagehand]
    command = "node"
    args = ["/absolute/path/to/stagehand/packages/integrations/core/dist/facade/stdio-server.mjs"]

    [mcp_servers.stagehand.env]
    STAGEHAND_BROWSER = "browserbase"
    BROWSERBASE_API_KEY = "bb_live_..."
    BROWSERBASE_PROJECT_ID = "..."
    `
Codex does not expand shell variables in config values; paste the real keys or generate this file from your environment. For a one-off run, pass the same values as `codex exec` overrides:

    `codex exec \
      -c mcp_servers.stagehand.command=node \
      -c 'mcp_servers.stagehand.args=["/absolute/path/to/packages/integrations/core/dist/facade/stdio-server.mjs"]' \
      -c 'mcp_servers.stagehand.env={ STAGEHAND_BROWSER = "browserbase", BROWSERBASE_API_KEY = "bb_live_..." }' \
      "your instruction"
    `
These overrides merge with any `[mcp_servers]` already in your `~/.codex/config.toml` rather than replacing them; set `CODEX_HOME` to a scratch directory if you need isolation.

`run` executes model-authored JavaScript in the browser. Use Browserbase for untrusted tasks and review the cite53†integration security boundary .
