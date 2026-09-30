# Shopify: About Shopify Markets

> Source: https://shopify.dev/docs/apps/build/markets
> Collected: 2026-09-30
> Published: Unknown

## Building an experience for customers

When building experiences for a merchant's customers, you should always query for and display the right content for the customer's language and country context. Shopify uses a defined fallback order when loading localized content. Consider a United States store with English as its primary language, which also sells in Belgium in French and Dutch. French is the default language for the shop's Belgian market.

When loading content for a customer browsing in a Dutch Belgian context, Shopify follows the following fallback order:

1. Dutch (Belgium)
2. Base Dutch
3. French (Belgium)
4. Base French
5. Base English (shop primary)

Shopify's buyer-facing APIs, including Liquid and the Storefront API, handle this fallback automatically.
