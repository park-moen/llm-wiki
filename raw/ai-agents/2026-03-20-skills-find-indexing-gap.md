# skills.sh find does not index two valid public skills

> Source: https://github.com/vercel-labs/skills/issues/705
> Collected: 2026-09-16
> Published: 2026-03-20

## Description

I verified two public skills repositories that are valid, installable, and already have live detail pages on `skills.sh`, but they still do not appear in `npx skills find`.

Affected repos:

- `GODGOD126/self-improving-for-codex`
- `GODGOD126/ai-hot-brief-telegram-public`

What I verified:

1. Both repos can be installed successfully with `npx skills add`.
2. Both repos have live detail pages returning HTTP 200:
   - `https://skills.sh/GODGOD126/self-improving-for-codex`
   - `https://skills.sh/GODGOD126/ai-hot-brief-telegram-public`
3. I repeated installs across multiple environments, including isolated home directories.
4. `npx skills find self-improving-for-codex` returns `No skills found`.
5. `npx skills find ai-hot-brief-telegram-public` returns `No skills found`.
6. Direct calls to the search API also return zero results:
   - `https://skills.sh/api/search?q=self-improving-for-codex&limit=10`
   - `https://skills.sh/api/search?q=ai-hot-brief-telegram-public&limit=10`

This suggests the issue is not with repo validity or installation, but with indexing / search ingestion on the platform side.

Questions:

1. Is there an undocumented delay, threshold, or manual review step before a skill becomes searchable via `find`?
2. Is a repo detail page on `skills.sh` expected to appear before search indexing is complete?
3. If manual action is required, could you help index these two skills?
