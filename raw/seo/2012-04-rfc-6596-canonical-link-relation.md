# The Canonical Link Relation

> Source: https://datatracker.ietf.org/doc/html/rfc6596
> Collected: 2026-09-08
> Published: 2012-04

## Abstract

RFC 5988 specifies a way to define relationships between links on the web. This document describes a new type of such a relationship, "canonical", to designate an Internationalized Resource Identifier (IRI) as preferred over resources with duplicative content.

This document is not an Internet Standards Track specification; it is published for informational purposes. It is a product of the Internet Engineering Task Force (IETF), represents the consensus of the IETF community, and was approved for publication by the Internet Engineering Steering Group (IESG).

## Introduction

The canonical link relation specifies the preferred IRI from resources with duplicative content. Common implementations specify the preferred version of an IRI from duplicate pages created with the addition of IRI parameters, such as session IDs, or specify the single-page version as preferred over the same content separated on multiple component pages.

Informally, "canonical" is the author's preferred version of a resource. More formally, the canonical link relation specifies the preferred IRI from a set of resources that return the context IRI's content in duplicated form. Once specified, applications such as search engines can focus processing on the canonical, and references to the context IRI can be updated to reference the target IRI.

## The Canonical Link Relation

The target (canonical) IRI MUST identify content that is either duplicative or a superset of the content at the context (referring) IRI. Authors who declare the canonical link relation ought to anticipate that applications such as search engines can:

- Index content only from the target IRI; content from the context IRIs will likely be disregarded as duplicative.
- Consolidate IRI properties, such as link popularity, to the target IRI.
- Display the target IRI as the representative IRI.

The target IRI MAY specify a relative IRI, be self-referential, exist on a different hostname or domain, have a different scheme name, or be a superset of the content at the context IRI.

Each component page of a multi-page article may specify a view-all version as the target IRI when the view-all version is a superset of their content. In contrast, page-2.html SHOULD NOT designate page-1.html as the target IRI: only content from page-1.html may be processed, and page-2.html content may be disregarded.

Administrators ought to specify only one canonical link relation for a resource. They should avoid designating as canonical a source IRI of a permanent redirect, an IRI that canonicalizes to another IRI, an IRI returning an error such as an HTTP 4xx response, or the first page of a multi-page article when it is not duplicative or a superset of the context IRI.

When the canonical link relation is declared improperly, such as a chained canonical or a target returning a 4xx response, applications can use their own heuristics and can ignore the improper canonical designation.

## Example

If the preferred version of content is:

```text
http://www.example.com/page.php?item=purse
```

duplicate content IRIs with added query parameters may declare it in HTML:

```html
<link rel="canonical"
      href="http://www.example.com/page.php?item=purse">
```

or in an HTTP header:

```http
Link: <http://www.example.com/page.php?item=purse>; rel="canonical"
```

## Recommendations

Before adding the canonical link relation, verification of the following is RECOMMENDED:

1. The content of the context IRI is duplicated within the content of the target IRI.
2. For HTTP, permanent HTTP redirects could not be implemented in place of the canonical link relation.
3. When the target IRI is a superset, the user experience is strongly considered, including possible increased load time and navigation complexity.
