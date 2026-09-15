# git-tag Documentation

> Source: https://git-scm.com/docs/git-tag.html
> Collected: 2026-08-10
> Published: Unknown

## Description

Add a tag reference in `refs/tags/`, unless `-d`/`-l`/`-v` is given to delete, list or verify tags.

Unless `-f` is given, the named tag must not yet exist.

Tag objects (created with `-a`, `-s`, or `-u`) are called "annotated" tags; they contain a creation date, the tagger name and e-mail, a tagging message, and an optional cryptographic signature. Whereas a "lightweight" tag is simply a name for an object (usually a commit object).

Annotated tags are meant for release while lightweight tags are meant for private or temporary object labels.

## Selected options

`-f`, `--force`

Replace an existing tag with the given name (instead of failing).

`-d`, `--delete`

Delete existing tags with the given names.

`--points-at` [<object>]

Only list tags of <object> (`HEAD` if not specified).

## On Re-tagging

If you never pushed anything out, just re-tag it. Use `-f` to replace the old one.

But if you have pushed things out (or others could just read your repository directly), then others will have already seen the old tag. In that case, just admit you screwed up, and use a different name.

People MUST be able to trust their tag-names.
