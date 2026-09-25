# Stagehand README

> Source: https://github.com/browserbase/stagehand
> Collected: 2026-09-25
> Published: Unknown

Stagehand is the SDK for browser agents

Docs · Quickstart · cite103†⭐ Star this repo L191: ## AI that uses the browser like humans.
Sign in once, keep the session, and pull structured data out the other side.

    import { localBrowser, Stagehand } from "@browserbasehq/stagehand";
    import { z } from "zod/v4";

    // Cookies persist in ./browser-data, so the next run starts already signed in
    const browser = await localBrowser.launch({ userDataDir: "./browser-data" });
    const stagehand = await Stagehand.create({
      browser,
      model: { modelName: "openai/gpt-5.4-mini", apiKey: process.env.OPENAI_API_KEY },
    });

    const [page] = await browser.context.pages();
    await page.goto("https://app.example.com/login");

    // observe() returns real selectors, so credentials never reach the model
    const { data: email } = await stagehand.observe("find the email input");
    const { data: password } = await stagehand.observe("find the password input");
    await page.locator(email[0].selector).fill(process.env.APP_EMAIL!);
    await page.locator(password[0].selector).fill(process.env.APP_PASSWORD!);

    // act() self-heals when the site redesigns its form
    await stagehand.act("click the sign in button");
    await stagehand.act("open the billing page");

    // extract() returns schema-validated data
    const { data } = await stagehand.extract(
      "extract every invoice in the table",
      z.object({
        invoices: z.array(z.object({ number: z.string(), amount: z.number(), paid: z.boolean() })),
      }),
    );

    console.log(data.invoices);

    await stagehand.close();
    await browser.close();
Python

    import asyncio
    import os

    from pydantic import BaseModel
    from stagehand import Stagehand, local_browser


    class Invoice(BaseModel):
        number: str
        amount: float
        paid: bool


    class Invoices(BaseModel):
        invoices: list[Invoice]


    async def main() -> None:
        # Cookies persist in ./browser-data, so the next run starts already signed in
        browser = await local_browser.launch(user_data_dir="./browser-data")
        try:
            stagehand = await Stagehand.create(
                browser=browser,
                model="openai/gpt-5.4-mini",
                model_api_key=os.environ["OPENAI_API_KEY"],
            )
            try:
                page = (await browser.context.pages())[0]
                await page.goto("https://app.example.com/login")
                # observe() returns real selectors, so credentials never reach the model
                email = await stagehand.observe("find the email input")
                password = await stagehand.observe("find the password input")
                await page.locator(email.data[0].selector).fill(os.environ["APP_EMAIL"])
                await page.locator(password.data[0].selector).fill(os.environ["APP_PASSWORD"])

                # act() self-heals when the site redesigns its form
                await stagehand.act("click the sign in button")
                await stagehand.act("open the billing page")

                # extract() returns schema-validated data
                result = await stagehand.extract(
                    "extract every invoice in the table",
                    Invoices,
                )
                print(result.data.invoices)
            finally:
                await stagehand.close()
        finally:
            await browser.close()
    asyncio.run(main())
Go

    package main

    import (
    	"context"
    	"errors"
    	"fmt"
    	"log"
    	"os"

    	stagehand "github.com/browserbase/stagehand/packages/sdk-go/v4"
    )

    type invoice struct {
    	Number string  `json:"number"`
    	Amount float64 `json:"amount"`
    	Paid   bool    `json:"paid"`
    }

    type invoices struct {
    	Invoices []invoice `json:"invoices"`
    }

    func main() {
