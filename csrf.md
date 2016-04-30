# Cross-Site Request Forgery (CSRF)

Because our library accepts data from HTML-based form posts, we need to have a
CSRF solution for any endpoint that we handle POST requests for.

POST requests to any endpoint that we handle, where the Content Type
of the request is `application/x-www-form-urlencoded` MUST be verified with a
CSRF token.

How this is achieved is not defined by our specification, for solutions please
refer to the [Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet][].

### Informational: Regarding JSON

If the Content Type of the POST request is `application/json`, it can be assumed
that this is not a CSRF attack because an HTML form cannot be maliciously
modified to produce this content type.

Note: the framework MUST NOT process a POST body as JSON if the Content-Type is 
not JSON, to avoid the text/plain workaround that is described in 16.3.1 CSRF 
protection and JSON.

Despite this, some frameworks still require `application/json` POSTS to supply
a token.  In these situations, we should use documentation to describe how 
clients can obtain token and submit it with the request.

[Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet]: https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet
