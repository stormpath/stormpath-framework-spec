<a name="#top">Back to Top</a>

# Sessions


## Table of Contents

* [Options](#Options)
  * [AUTO_LOGIN](#AUTO_LOGIN)
  * [ENABLE_LOGIN](#ENABLE_LOGIN)
  * [REDIRECT_URL](#REDIRECT_URL)
  * [LOGIN_URL](#LOGIN_URL)
  * [GOOGLE_CALLBACK_URL](#GOOGLE_CALLBACK_URL)
  * [FACEBOOK_CALLBACK_URL](#FACEBOOK_CALLBACK_URL)
  * [GITHUB_CALLBACK_URL](#GITHUB_CALLBACK_URL)
  * [LINKEDIN_CALLBACK_URL](#LINKEDIN_CALLBACK_URL)
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
    
  }
}
```


#### <a name="AUTO_REDIRECT"></a> AUTO_REDIRECT

If enabled, and a user already has a valid session, instead of re-rendering the
login page we will redirect this user to the URL specified by `REDIRECT_URL`.

If disabled, we won't redirect the user anywhere and will simply re-render the
login page.  However -- in this case we will *also* destroy any existing user
sessions.  This ensures that odd edge cases won't come up wherein a user is
viewing a login page but can see their account information in some place (like a
menu bar).

<a href="#top">Back to Top</a>


#### <a name="ENABLE_LOGIN"></a> ENABLE_LOGIN

If `True` this feature will be enabled and our library will intercept requests
at the [LOGIN_URL](#LOGIN_URL).  When the application server starts, we will
query the user's Stormpath Application to discover what Account Store Mappings
are available.  We will then pre-load these so that the login page displays all
available forms of login (*this might include username / email and password
login, Facebook Login, Google Login, etc.*).

If `False` this feature is disabled and the base framework will be responsible
for the [LOGIN_URL](#LOGIN_URL), likely resulting in a 404 Not Found error.

**NOTE**: If this feature is enabled, and no Account Stores are mapped to this
Application -- we will throw an error during initialization since there are no
possible ways for a user to authenticate.


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
