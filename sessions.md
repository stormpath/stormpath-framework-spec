<a name="#top">Back to Top</a>

# Sessions


## Table of Contents

* [Options](#Options)
  * [ttl](#ttl)
  * [tti](#tti)
* [Incoming Request Handling](#incoming-request-handling)


## Feature Description

This document describes the how our framework integrations should handle user
sessions and session management.

There are a few things we'll do in respect to user sessions:

- We'll always store a signed JSON Web Token (JWT) in an unencrypted, unsigned
  browser cookie.  If a user inspects their cookies in their browser, they
  should be able to see the JWT directly.
- This cookie will always be set with the `HttpOnly` flag set, meaning that no
  Javascript access will be allowed to this cookie.
- This cookie will be set with the `secure` flag if the application is in
  production -- this ensures that no cookies will be set over plain old HTTP
  (*leaking information to potential attackers*).
- The cookie will always be named `access_token`.
- The cookie will have a set `ttl` and `tti` which are developer-configurable.
  NOTE: The `ttl` is the absolute expiration time of the JWT, while the `tti` is
  the absolute expiration of the cookie itself.

**NOTE**: We do NOT need to support server-side session storage, because the way
this works is like so: a JWT is created which contains an account href.  The
account href is persisted in our internal Stormpath SDK cache.  This means no
server side session storage is necessary as the caching optimizations are
already handled internally (*by us*).


## <a name="Options"></a> Options

This table is a list of all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (e.g. to camel case, or not)? Is not specified here.

```json
{
  "session": {
    "ttl": 3600,
    "tti": 900
    "httpOnly": true,
    "secure": null,
    "path": "/",
    "domain": null,
  }
}
```


#### <a name="ttl"></a> ttl

This option represents the maximum amount of time (*in seconds*) that a user
may remain logged in via a session.

Once this time has passed, a new session will need to be created.

When a user first authenticates to the website, this setting's value will be
used as the JWT's expiration timestamp.

**NOTE**: If this value is set to `0`, this is a special case.  Our library will
set the `ttl` value to ten years in the future (*essentially, forever*).

<a href="#top">Back to Top</a>


#### <a name="tti"></a> tti

This option represents the maximum amount of time (*in seconds*) that a user
may remain logged in via a session *before the session's length is extended*.

Once this time has passed, a new session will need to be created.

Each time an authenticated user makes a request to the website, the `tti` value
will be used to update the cookie's expiration time by this amount.  This lets
you build websites where unless a user is inactive (*or the maximum session ttl
has been reached*), a user can continue browsing the site with an active session
indefinitely.

**NOTE**: If this value is set to `0`, this is a special case.  Our library will
set the `tti` value to 10 years in the future (*essentially, forever*).

<a href="#top">Back to Top</a>


## <a name="incoming-request-handling"></a> Incoming Request Handling

When a request is made to a user's web application, our Stormpath libraries
must:

- Intercept the incoming request before it reaches the developer's code.
- Inspect the cookies, and look for an `access_token` cookie.
- If the `access_token` cookie is present, we should retrieve the value inside
  as a string, and decode this as a JWT.
- If the JWT decoding fails for any reason (token is expired, invalid signature,
  etc.) -- we should reject the request with a 401 UNAUTHORIZED and no body.
- If the JWT is valid, we should retrieve the `href` property from the JWT, and
  retrieve the user's account using the `href` property directly.  We should
  then make this account object available to the developer's code so the
  developer can recognize the user making the request.

<a href="#top">Back to Top</a>
