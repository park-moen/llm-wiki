# Stagehand README: Install and Usage

> Source: https://github.com/browserbase/stagehand
> Collected: 2026-09-25
> Published: Unknown

## Install

    pnpm add @browserbasehq/stagehand 'zod@~4.4.3'

Python

    pip install stagehand

Go

    go get github.com/browserbase/stagehand/packages/sdk-go/v4@v4.0.0

Local runs need Chrome installed. Full setup: Quickstart .
## Why Stagehand
Familiar APIs  | The Playwright-style methods you and your agents already know: `goto`, `click`, `locator`, `screenshot`.
Token efficiency  | Hybrid accessibility-tree trimming gives agents exactly the page context they need and nothing more.
Faster in production  | Stagehand runs as an extension next to the browser, cutting round-trip latency on every action.
Self-healing  | `act`, `observe`, and `extract` refresh how an action happens when the site changes underneath it.
Built for agents  | WebMCP, clipboard support, batch commands, deep locators for nested iframes and closed Shadow DOMs, OTel traces.
Three languages  | One complete browser driver across TypeScript, Python, and Go.
## Run it on Browserbase
Point the same script at Browserbase and get 2x faster execution than Playwright cloud equivalent browsers. Configure the Model Gateway so you never wire up a provider, and enable server-side caching to cache repeated actions.

    import { browserbase, Stagehand } from "@browserbasehq/stagehand";

    const browser = await browserbase.launch({ apiKey: process.env.BROWSERBASE_API_KEY! });
    // No model configuration: the Model Gateway picks the cheapest model for each action
    // cache: true: identical calls come back from Browserbase, no tokens spent
    const stagehand = await Stagehand.create({ browser, cache: true });
Python

    import os

    from stagehand import Stagehand, browserbase

    browser = await browserbase.launch(api_key=os.environ["BROWSERBASE_API_KEY"])

    # No model configuration: the Model Gateway picks the cheapest model for each action
    # cache=True: identical calls come back from Browserbase, no tokens spent
    stagehand = await Stagehand.create(browser=browser, cache=True)
Go

    browser, err := stagehand.LaunchBrowserbase(ctx, stagehand.BrowserbaseLaunchOptions{
    	APIKey: os.Getenv("BROWSERBASE_API_KEY"),
    })
    if err != nil {
    	return err
    }

    // No model configuration: the Model Gateway picks the cheapest model for each action
    // CacheEnabled(true): identical calls come back from Browserbase, no tokens spent
    cache := stagehand.CacheEnabled(true)
    client, err := stagehand.Create(ctx, stagehand.CreateOptions{
    	Browser: browser,
    	Cache:   &cache,
    })
    if err != nil {
    	return err
    }
Verified mode , residential proxies , persistent contexts , and session recordings come with it. Get an API key and learn how to configure your browser here .
## Give your coding agent a browser

The hosted Browserbase MCP server puts `navigate`, `act`, `observe`, and `extract` in any MCP client — no install, no local browser.

    claude mcp add --transport http browserbase https://mcp.browserbase.com/mcp \
      --header "Authorization: Bearer $BROWSERBASE_API_KEY"
Cursor, Codex, and other MCP clients

    {
      "mcpServers": {
        "browserbase": {
          "url": "https://mcp.browserbase.com/mcp",
          "headers": { "Authorization": "Bearer YOUR_BROWSERBASE_API_KEY" }
        }
      }
    }
## Search and fetch without a browser
Fetch lets you grab the content of any URL as markdown. Search provides fast, token-efficient web search results. Both as a lightweight complement to browser sessions.

    import { browserbase } from "@browserbasehq/stagehand";

    const { results } = await browserbase.search({
      apiKey: process.env.BROWSERBASE_API_KEY!,
      query: "browser agent frameworks",
      numResults: 5,
    });

    const fetched = await browserbase.fetch({
      apiKey: process.env.BROWSERBASE_API_KEY!,
      url: results[0].url,
      format: "markdown",
    });

    console.log(fetched.content);
## Docs and resources
Quickstart | Empty directory to working automation in three steps
page · locator | Playwright-style browser and element APIs
WebMCP | Discover and invoke WebMCP tools exposed by web pages
act · extract · observe | Browser actions and data extraction with natural language
Search · Fetch | Web search and page content without a browser
Migrate from Playwright | Port an existing suite
Integrations | CrewAI, Mastra, Deep Agents, Vercel AI SDK, Claude Code, Codex
Python SDK · cite126 | Language-specific guides
Ask DeepWiki | Ask questions about this codebase
## Join the community

Stagehand is built in the open, and the fastest way to shape it is to show up.

  * ⭐ Star this repo — it is how most people find Stagehand
  * cite128 — questions, support, and what we are building next
  * 🐛 Open an issue — bug reports are the most useful contribution
  * cite129 — releases and demos
### Contributing

We're focused on improving reliability, extensibility, speed, and cost, in that order. Bug fixes and small improvements are the best way to get started. For anything larger, reach out to Miguel Gonzalez or Paul Klein on Discord first so we can make sure it lands.
Stagehand is a TypeScript, Python, and Go monorepo driven by `just` :

    git clone https://github.com/browserbase/stagehand.git
    cd stagehand
    just install
    just generate
    just build

    export OPENAI_API_KEY="your-openai-api-key"
    just example act # runs packages/sdk-ts/examples/act.ts

See cite86 for the full TypeScript, Python, and Go setup.
## Acknowledgements

We'd like to thank the following people for their major contributions to Stagehand:

  * Paul Klein L519:   * cite134 L520:   * Miguel Gonzalez L521:   * cite136 L522:   * Thomas Katwan L523:   * cite138 L524:   * Anirudh Kamath L525:   * cite140 L526:   * Navid Pour L527:   * cite142 L528:   * Sam Finton L529:   * cite144 L530:   * Shriya Lolabattu L531:   * cite146 L532: ## License

Licensed under the MIT License.

Copyright 2026 Browserbase, Inc.

"Stagehand" is a trademark of Browserbase, Inc.
