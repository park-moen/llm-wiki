# Search relevance: exact skill name matches are not prioritized over description keywords

> Source: https://github.com/vercel-labs/skills/issues/1303
> Collected: 2026-09-16
> Published: 2026-05-29

## Current Behavior

When searching for a skill by its exact name (e.g., `jscpd`), the search results rank skills that merely mention the query deep inside their description higher than the skill whose actual repository/name is the query itself.

In my case, searching for `jscpd` returns other skills that reference `jscpd` in their body text, while the actual jscpd skill (from the `jscpd` repository) appears lower in the results and is not the top match.

## Suggested Solution

Consider updating the search ranking algorithm to prioritize:

1. Exact matches on the skill name / repository name.
2. Prefix matches (e.g., repo name starts with the query).
3. Description and keyword matches as a secondary signal.

This would make search results more relevant and help users find specific tools quickly.

## Additional Context

It appears that some skill descriptions mention `jscpd` as an example of a tool the authors do not use. Because the current search seems to scan all text equally (without boosting the canonical name/repo), it surfaces these tangential mentions above the actual skill, making the results feel noisy and inaccurate.

## Expected Behavior

An exact or near-exact match on the skill name or repository name should be heavily weighted and appear as the first result. Description text matches should be secondary.
