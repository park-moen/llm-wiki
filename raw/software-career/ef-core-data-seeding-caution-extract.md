# Data Seeding — selected caution

> Source: https://learn.microsoft.com/en-us/ef/core/modeling/data-seeding
> Collected: 2026-09-17
> Published: Unknown

## Selected source text

> The seeding code should not be part of the normal app execution as this can cause concurrency issues when multiple instances are running.
