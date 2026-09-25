# Stagehand Quickstart (excerpt)

> Source: https://docs.stagehand.dev/v4/first-steps/quickstart
> Collected: 2026-09-25
> Published: Unknown

# Quickstart

Build your first Stagehand automation with act, extract, and observe.

Copy page Copy page

The quickest way to start with Stagehand is to install the SDK, point it at a browser, and write a script. This page gets you from an empty directory to a working automation in three steps, using a browser on your own machine.

1

Create a sample project

  * TypeScript

  * Python
  * Go

    `mkdir my-stagehand-app && cd my-stagehand-app
    pnpm init -y
    pnpm install @browserbasehq/stagehand 'zod@~4.4.3'
    `
Keep Zod on the `4.4.x` minor version to match Stagehand’s supported types. Newer Zod minor versions can cause TypeScript errors when passing schemas to `extract()`.

    `mkdir my-stagehand-app && cd my-stagehand-app
    python3 -m venv .venv && source .venv/bin/activate
    pip install stagehand
    `

    `mkdir my-stagehand-app && cd my-stagehand-app
    go mod init example.com/my-stagehand-app
    go get github.com/browserbase/stagehand/packages/sdk-go/v4@v4.0.0
    `
Stagehand drives a local browser through the Chrome DevTools Protocol, so install cite48†Chrome†www.google.com on your machine before running the script.

2

Write the script

Create the example script (`index.ts`, `main.py`, or `main.go`). It exercises all three primitives: act, extract, and observe.

  * TypeScript

  * Python
  * Go

    `import { localBrowser, Stagehand } from "@browserbasehq/stagehand";
    import { z } from "zod/v4";

    async function main() {
      const browser = await localBrowser.launch();
      try {
        const stagehand = await Stagehand.create({
          browser,
          model: {
            modelName: "openai/gpt-5.6-sol",
            apiKey: process.env.OPENAI_API_KEY,
          },
        });
        console.log("Stagehand session started");
        try {
          const [page] = await browser.context.pages();

          await page.goto("https://stagehand.dev");

          const extractResult = await stagehand.extract(
            "Extract the value proposition from the page.",
            z.object({ valueProposition: z.string() }),
          );
          console.log("Extract result:\n", extractResult.data);

          await stagehand.act("Click the 'Evals' button.");
          const observeResult = await stagehand.observe("What can I click on this page?");
          console.log("Observe result:\n", observeResult.data);
        } finally {
          await stagehand.close();
        }
      } finally {
        await browser.close();
      }
    }

    main().catch((err) => {
      console.error(err);
      process.exit(1);
    });
    `
Stagehand never reads environment variables on your behalf. Read your model provider API key in your own code and pass it to `Stagehand.create()`, as the script above does. A local browser cannot use the cite49†Model Gateway , so Stagehand requires you to provide a model and its API key.

3

Run it

Set your model provider API key, then run the script. Stagehand launches Chrome on your machine and opens a visible window, so you can watch each step as it happens.

  * TypeScript

  * Python
  * Go

    `export OPENAI_API_KEY="sk-..." # Your model provider API key
    pnpm dlx tsx index.ts          # Run the example script
    `

    `export OPENAI_API_KEY="sk-..." # Your model provider API key
    python main.py                 # Run the example script
    `

    `export OPENAI_API_KEY="sk-..." # Your model provider API key
    go run .                       # Run the example script
    `
Ready to run in the cloud, with stealth, proxies, and session recordings? Swap `localBrowser.launch()` for `browserbase.launch()` with your Browserbase API key, and the cite49†Model Gateway picks and authenticates a model for you. See cite14†Browser configuration .
