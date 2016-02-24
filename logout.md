# Logout

The integration must implement the `/logout` endpoint.  The endpoint is used to
delete the [Authentication Cookies](cookie-authentication.md) that were set on
login.

This endpoint should respond to POST requests only.  Responding to GET requests
is problematic because the browser's Omnibar can make arbitrary GET requests to
this endpoint, and Robert can troll you with superlogout-dot-com.  For more
information please see
[Logout: GET or POST?](http://stackoverflow.com/questions/3521290/logout-get-or-post)

If ID Site is enabled with `stormpath.web.idSite.enabled`, the user should also
be redirected through ID site after deleting the local cookies.  This should be
done with the ID Site URL Builder in the SDK, and setting the `logout: true`
option.  This needs to happen because we maintain a cookie at
`api.stormpath.com/sso` which needs to be removed.