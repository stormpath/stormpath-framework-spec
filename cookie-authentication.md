<a name="#top">Back to Top</a>

# Cookie Authentication

When a user authenticates with a web browser, we need to persist that authentication
result so that the user does not have to present their credentials on every request.
To achieve this we will use our OAuth2 password grant flow, and store the
tokens in cookies.

**NOTE**: This is *not* a "session" implementation, there is no server-side
state being managed for the developer.  This is pure authentication, using
cookies as the secure storage location on the client.

### Authentication Flow

1. The user visits the login page and submits their login and password.

2. The integration uses the SDK to perform the password grant exchange, using
    the Stormpath Application's `/oauth/token` endpoint.  This means that the login route is going to be using the SDK of choice to interact with the /v1/applications/:appId/oauth/token endpoint.

3. The integration collects the access token and refresh token from the SDK
    response in the previous step.  It then responds to the browser request by
    settings these values in cookies.

4. Subsequent requests by the browser will supply these cookies which contain the
    access and refresh tokens.  The integration should perform the following
    logic, using the SDK's relevant JWT and OAuth authenticators:

  * If the access token is valid, authenticate the request

  * If the access token is invalid or missing, attempt to use the refresh token to get a
  new access token.

  * If a new access token is obtained, authenticate the request and store this
  new token value in the access token cookie.

  * If a new access token cannot be obtained, reject the request and delete the
    refresh token cookie and the access token cookie.

### Rejecting Requests

If the request cannot be authenticated, how to respond will depend on the
content preference of the request:

* `text/html` - respond with a 302 redirect to the login view, and preserve the current URL so the user can be redirected to their original destination after login. If possible, use built-in features of the host framework to do this securely; otherwise, use a secure method of passing the destination URL forward.

* `application/json` - respond with `401 Unauthorized` and an empty body.

### Cookie Flags

For each cookie, access token and refresh token, use these rules to determine
the flags to set on the cookies when you are sending them to the browser:

* The `expires` flag should be determined by the `ttl` value for the
token, as defined by the Oauth Policy of the Stormpath Application.

* The `HttpOnly` flag should be set as follows:

  * If `stormpath.web.[accessTokenCookie|refreshTokenCookie].httpOnly` is false, do not set this flag
  * Othwerwise, set this flag to true

* The `path` flag should be set as follows:

  If `stormpath.web.[accessTokenCookie|refreshTokenCookie].path` has a non-null
  value, use that value as the cookie path. If the value is undefined or null,
  default to the web application framework default value. If the framework does
  not provide a default value, fall back to `stormpath.web.basePath` if defined.
  If `stormpath.web.basePath` is undefined or null, fallback to `/`.

* The `domain` flag should be set as follows:

  * If `stormpath.web.[accessTokenCookie|refreshTokenCookie].domain` has a
  non-null value, use that value.  Otherwise, fallback to the web framework
  domain value (if it defines one).  Finally, fallback to "" as the value.

* The `secure` flag should be set as follows:

  * If `stormpath.web.[accessTokenCookie|refreshTokenCookie].secure` is defined, use that value
  * Othwerwise, the framework should make a best effort to determine if the
    incoming request is HTTPS and set the `secure` flag if so.

## <a name="Configuration Options"></a> Configuration Options

The developer has control over the name of the cookie and the flags that
control the cookie behavior in the browser.  The lifetime of the cookies is
controlled by the application's OAuth Policy, and as such is not represented
in these option blocks.

```yaml
web:
  accessTokenCookie:
    name: "access_token"
    httpOnly: true
    secure: null
    path: null
    domain: null
  refreshTokenCookie:
    name: "refresh_token"
    httpOnly: true
    secure: null
    path: null
    domain: null
```

#### <a name="http-only"></a> httpOnly

This option determines whether or not this cookie can be read by JavaScript code
on the browser.  By default this is true, as this is secure.

**NOTE**: Disabling this option is a potential security issue, as your access
tokens could be hijacked via XSS.

<a href="#top">Back to Top</a>


#### <a name="secure"></a> secure

This flag informs the browser what types of transport can be used for sending
the cookie.  If this option is `true`, then the browser will only send the
cookie if the connection to the server is using HTTPS.

This value is null by default, which means that either (a) the developer must
explicitly set this value or (b) the framework should make a best effort attempt
to determine if the incoming request is occurring over an HTTPS connection.

It is our recommendation that the developer define this value as `true`, via
configuration, in their production environment.

**NOTE**: Setting this option to false is a security problem, as sending
cookies over a non-https connection is vulnerable to man-in-the-middle attacks.

<a href="#top">Back to Top</a>


#### <a name="path"></a> path

This option determines what URI paths will be able to read the cookie.

<a href="#top">Back to Top</a>


#### <a name="domain"></a> domain

This option determines what domain(s), can read this cookie value.

<a href="#top">Back to Top</a>

