# Ruby on Rails Guides: Internationalization (I18n)

> Source: https://guides.rubyonrails.org/i18n.html
> Collected: 2026-09-30
> Published: Unknown

## Internationalization and Localization

Internationalization (I18n) is the process of preparing your application to support multiple languages and regional formats. In practice, this usually means abstracting strings and other locale-specific elements, such as date or currency formats, out of your application.

Localization (L10n) is the process of providing translations and locale-specific formats for those abstracted pieces.

## How I18n in Ruby on Rails Works

The I18n API is primarily intended for translating user-facing text within the application. Translating model content itself (for example, blog posts stored in the database) requires a separate approach.

It is possible to swap the shipped `Simple` backend with a more powerful one, which would store translation data in a relational database, GetText dictionary, or similar.
