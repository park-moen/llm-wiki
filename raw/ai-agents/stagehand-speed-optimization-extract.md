# Stagehand Speed optimization (excerpt)

> Source: https://docs.stagehand.dev/v4/best-practices/speed-optimization
> Collected: 2026-09-25
> Published: Unknown

Best practices
# Speed optimization

Optimize Stagehand performance for faster automation and reduced latency

Copy page Copy page

Stagehand performance depends on several factors: DOM processing speed, LLM inference time, browser operations, and network latency. This guide provides proven strategies to maximize automation speed.

##

cite48†​ L101: 
Quick performance wins
###

cite49†​ L106: 
Plan ahead with observe

Use a single `observe()` call to plan multiple actions, then replay each returned `Action` through `act()`:
    `// Instead of sequential operations with multiple LLM calls
    await stagehand.act("Fill name field");         // LLM call #1
    await stagehand.act("Fill email field");        // LLM call #2
    await stagehand.act("Select country dropdown"); // LLM call #3

    // Use a single observe to plan all form fields: one LLM call
    const { data: formFields } = await stagehand.observe("Find all form fields to fill");

    // Replay each observed action: no further LLM inference
    for (const field of formFields) {
      await stagehand.act(field);
    }
    `
Performance tip: Passing an observed `Action` back to `act()` replays its recorded method and arguments without another inference call. A planned workflow therefore costs one `observe()` inference instead of one per step, which is the recommended pattern for multi-step workflows.
## Caching guide

Learn advanced caching patterns and cache invalidation strategies
###

cite50†​ L169: 
Optimize DOM processing

Reduce DOM complexity before Stagehand processes the page. Scoping is usually the biggest win: pass a locator so Stagehand snapshots one container instead of the whole document, and prune known-noisy subtrees.
Performance monitoring and benchmarking

Track performance metrics and measure optimization impact:
