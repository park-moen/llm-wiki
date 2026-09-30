# Wagtail Documentation: Internationalization

> Source: https://docs.wagtail.org/en/stable/advanced_topics/i18n.html
> Collected: 2026-09-30
> Published: Unknown

## How locales and translations are recorded in the database

All pages (and any snippets that have translation enabled) have a `locale` and `translation_key` field:

- `locale` is a foreign key to the `Locale` model
- `translation_key` is a UUID that’s used to find translations of a piece of content. Translations of the same page/snippet share the same value in this field

These two fields have a ‘unique together’ constraint so you can’t have more than one translation in the same locale.

## Translated homepages

If Wagtail can’t find a homepage that matches the user’s language, it will fall back to the page that is selected as the ‘root page’ on the site record, so you can use this field to specify the default language of your site.
