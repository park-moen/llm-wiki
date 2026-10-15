# RFC 9110: HTTP Semantics — redirection and 307

> Source: https://www.rfc-editor.org/rfc/rfc9110.txt
> Collected: 2026-10-13
> Published: Unknown
> Publication note: The RFC states June 2022; an exact publication day is not specified.
> Scope: Verbatim excerpts of Copyright Notice, Location, Redirection 3xx, and 307 Temporary Redirect; remaining sections omitted.

```text
Copyright Notice

   Copyright (c) 2022 IETF Trust and the persons identified as the
   document authors.  All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal
   Provisions Relating to IETF Documents
   (https://trustee.ietf.org/license-info) in effect on the date of
   publication of this document.  Please review these documents
   carefully, as they describe your rights and restrictions with respect
   to this document.  Code Components extracted from this document must
   include Revised BSD License text as described in Section 4.e of the
   Trust Legal Provisions and are provided without warranty as described
   in the Revised BSD License.

   This document may contain material from IETF Documents or IETF
   Contributions published or made publicly available before November
   10, 2008.  The person(s) controlling the copyright in some of this
   material may not have granted the IETF Trust the right to allow
   modifications of such material outside the IETF Standards Process.
   Without obtaining an adequate license from the person(s) controlling
   the copyright in such materials, this document may not be modified
   outside the IETF Standards Process, and derivative works of it may
   not be created outside the IETF Standards Process, except to format
   it for publication as an RFC or to translate it into languages other
   than English.


10.2.2.  Location

   The "Location" header field is used in some responses to refer to a
   specific resource in relation to the response.  The type of
   relationship is defined by the combination of request method and
   status code semantics.

     Location = URI-reference

   The field value consists of a single URI-reference.  When it has the
   form of a relative reference ([URI], Section 4.2), the final value is
   computed by resolving it against the target URI ([URI], Section 5).

   For 201 (Created) responses, the Location value refers to the primary
   resource created by the request.  For 3xx (Redirection) responses,
   the Location value refers to the preferred target resource for
   automatically redirecting the request.

   If the Location value provided in a 3xx (Redirection) response does
   not have a fragment component, a user agent MUST process the
   redirection as if the value inherits the fragment component of the
   URI reference used to generate the target URI (i.e., the redirection
   inherits the original reference's fragment, if any).

   For example, a GET request generated for the URI reference
   "http://www.example.org/~tim" might result in a 303 (See Other)
   response containing the header field:

   Location: /People.html#tim

   which suggests that the user agent redirect to
   "http://www.example.org/People.html#tim"

   Likewise, a GET request generated for the URI reference
   "http://www.example.org/index.html#larry" might result in a 301
   (Moved Permanently) response containing the header field:

   Location: http://www.example.net/index.html

   which suggests that the user agent redirect to
   "http://www.example.net/index.html#larry", preserving the original
   fragment identifier.

   There are circumstances in which a fragment identifier in a Location
   value would not be appropriate.  For example, the Location header
   field in a 201 (Created) response is supposed to provide a URI that
   is specific to the created resource.

      |  *Note:* Some recipients attempt to recover from Location header
      |  fields that are not valid URI references.  This specification
      |  does not mandate or define such processing, but does allow it
      |  for the sake of robustness.  A Location field value cannot
      |  allow a list of members because the comma list separator is a
      |  valid data character within a URI-reference.  If an invalid
      |  message is sent with multiple Location field lines, a recipient
      |  along the path might combine those field lines into one value.
      |  Recovery of a valid Location field value from that situation is
      |  difficult and not interoperable across implementations.

      |  *Note:* The Content-Location header field (Section 8.7) differs
      |  from Location in that the Content-Location refers to the most
      |  specific resource corresponding to the enclosed representation.
      |  It is therefore possible for a response to contain both the
      |  Location and Content-Location header fields.


15.4.  Redirection 3xx

   The 3xx (Redirection) class of status code indicates that further
   action needs to be taken by the user agent in order to fulfill the
   request.  There are several types of redirects:

   1.  Redirects that indicate this resource might be available at a
       different URI, as provided by the Location header field, as in
       the status codes 301 (Moved Permanently), 302 (Found), 307
       (Temporary Redirect), and 308 (Permanent Redirect).

   2.  Redirection that offers a choice among matching resources capable
       of representing this resource, as in the 300 (Multiple Choices)
       status code.

   3.  Redirection to a different resource, identified by the Location
       header field, that can represent an indirect response to the
       request, as in the 303 (See Other) status code.

   4.  Redirection to a previously stored result, as in the 304 (Not
       Modified) status code.

      |  *Note:* In HTTP/1.0, the status codes 301 (Moved Permanently)
      |  and 302 (Found) were originally defined as method-preserving
      |  ([HTTP/1.0], Section 9.3) to match their implementation at
      |  CERN; 303 (See Other) was defined for a redirection that
      |  changed its method to GET.  However, early user agents split on
      |  whether to redirect POST requests as POST (according to then-
      |  current specification) or as GET (the safer alternative when
      |  redirected to a different site).  Prevailing practice
      |  eventually converged on changing the method to GET.  307
      |  (Temporary Redirect) and 308 (Permanent Redirect) [RFC7538]
      |  were later added to unambiguously indicate method-preserving
      |  redirects, and status codes 301 and 302 have been adjusted to
      |  allow a POST request to be redirected as GET.

   If a Location header field (Section 10.2.2) is provided, the user
   agent MAY automatically redirect its request to the URI referenced by
   the Location field value, even if the specific status code is not
   understood.  Automatic redirection needs to be done with care for
   methods not known to be safe, as defined in Section 9.2.1, since the
   user might not wish to redirect an unsafe request.

   When automatically following a redirected request, the user agent
   SHOULD resend the original request message with the following
   modifications:

   1.  Replace the target URI with the URI referenced by the redirection
       response's Location header field value after resolving it
       relative to the original request's target URI.

   2.  Remove header fields that were automatically generated by the
       implementation, replacing them with updated values as appropriate
       to the new request.  This includes:

       1.  Connection-specific header fields (see Section 7.6.1),

       2.  Header fields specific to the client's proxy configuration,
           including (but not limited to) Proxy-Authorization,

       3.  Origin-specific header fields (if any), including (but not
           limited to) Host,

       4.  Validating header fields that were added by the
           implementation's cache (e.g., If-None-Match,
           If-Modified-Since), and

       5.  Resource-specific header fields, including (but not limited
           to) Referer, Origin, Authorization, and Cookie.

   3.  Consider removing header fields that were not automatically
       generated by the implementation (i.e., those present in the
       request because they were added by the calling context) where
       there are security implications; this includes but is not limited
       to Authorization and Cookie.

   4.  Change the request method according to the redirecting status
       code's semantics, if applicable.

   5.  If the request method has been changed to GET or HEAD, remove
       content-specific header fields, including (but not limited to)
       Content-Encoding, Content-Language, Content-Location,
       Content-Type, Content-Length, Digest, Last-Modified.

   A client SHOULD detect and intervene in cyclical redirections (i.e.,
   "infinite" redirection loops).

      |  *Note:* An earlier version of this specification recommended a
      |  maximum of five redirections ([RFC2068], Section 10.3).
      |  Content developers need to be aware that some clients might
      |  implement such a fixed limitation.


15.4.8.  307 Temporary Redirect

   The 307 (Temporary Redirect) status code indicates that the target
   resource resides temporarily under a different URI and the user agent
   MUST NOT change the request method if it performs an automatic
   redirection to that URI.  Since the redirection can change over time,
   the client ought to continue using the original target URI for future
   requests.

   The server SHOULD generate a Location header field in the response
   containing a URI reference for the different URI.  The user agent MAY
   use the Location field value for automatic redirection.  The server's
   response content usually contains a short hypertext note with a
   hyperlink to the different URI(s).

```
