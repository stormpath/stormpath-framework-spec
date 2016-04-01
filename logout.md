# Logout

The integration must implement the `/logout` endpoint.  The endpoint is used to
delete the [Authentication Cookies](cookie-authentication.md) that were set on
login.  The access and refresh tokens that were issued (and stored in cookies)
must also be deleted from the Stormpath REST API.

This endpoint should respond to POST requests only.  Responding to GET requests
is problematic because the browser's Omnibar can make arbitrary GET requests to
this endpoint, and Robert can troll you with superlogout-dot-com.  For more
information please see
[Logout: GET or POST?](http://stackoverflow.com/questions/3521290/logout-get-or-post)

The response depends on the request preference:

* `text/html` should redirect to `stormpath.web.logout.nextUri`.

* `application/json` should respond with `200 OK` and an empty body.

If ID Site is enabled with `stormpath.web.idSite.enabled`, the user should also
be redirected through ID site after deleting the local cookies and deleting the
tokens.  This should be done with the ID Site URL Builder in the SDK, by
setting the `logout: true` option when building a URL.  This needs to happen
because we maintain a cookie at `api.stormpath.com/sso` which needs to be
removed.
