# Shopify GraphQL Admin: Translation

> Source: https://shopify.dev/docs/api/admin-graphql/latest/objects/translation
> Collected: 2026-09-30
> Published: Unknown

## Translation

A localized version of a field on a resource. Translations enable merchants to provide content in multiple languages for `Product` objects, `Collection` objects, and other store resources.

Each translation specifies the locale, the field being translated (identified by its key), and the translated value. Translations can be market-specific, allowing different content for the same language across different markets, or available globally when no `Market` is specified. The `outdated` flag indicates whether the original content has changed since this translation was last updated.

## outdated

Whether the original content has changed since this translation was updated.
