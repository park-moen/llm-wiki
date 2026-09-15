# Canonical link element

> Source: https://en.wikipedia.org/wiki/Canonical_link_element
> Collected: 2026-09-08
> Published: Unknown

A canonical link element is an HTML element that helps webmasters prevent duplicate content issues in search engine optimization by specifying the "canonical" or "preferred" version of a web page. It is described in RFC 6596, which went live in April 2012.

## Purpose

Content duplication can happen through GET parameters, multiple URLs from a CMS such as www or non-www URLs, access through different hosts or protocols such as HTTPS or HTTP, and print versions of websites. Duplicate content issues occur when the same content is accessible from multiple URLs.

The canonical link element can be inserted into the `<head>` section of a web page. It helps webmasters make clear to search engines which page should be credited as the original and lets search engines consolidate indexing and ranking signals from duplicate or similar pages into a single preferred URL.

## Implementation

The canonical link element can be used in the semantic HTML `<head>` or sent with the HTTP header of a document. For non-HTML documents, the HTTP header is an alternate way to set a canonical URL.

```html
<link rel="canonical" href="http://example.com/">
```

```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Link: <https://www.newthink.life/page.php>; rel="canonical"
Content-Length: 4223
```

Last edited: 27 August 2026, at 11:53 (UTC).
