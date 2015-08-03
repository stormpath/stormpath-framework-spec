<a name="#top">Back to Top</a>

# Sessions


## Table of Contents

* [Options](#Options)
  * [httpOnly](#http-only)
  * [secure](#secure)
  * [path](#path)
  * [domain](#domain)
  * [validationStrategy](#validation-strategy)
* [Incoming Request Handling](#incoming-request-handling)
* [/me Handling](#me-handling)


## Feature Description

This document describes the how our framework integrations should handle user
sessions and session management.

There are a few things we'll do in respect to user sessions:

- We'll always store an access and refresh token in the browser,
these tokens will be obtained by using the password grant flow on
the configured Stormpath application

- This cookie will always be set with the `HttpOnly` flag set, meaning that no
  Javascript access will be allowed to this cookie.

- This cookie will be set with the `secure` flag if the application is in
  production -- this ensures that no cookies will be set over plain old HTTP
  (*leaking information to potential attackers*).

- The cookies will always be named `accessToken` and `refreshToken`.

- The Expiration time each cookie should be determed by the `exp` claim of
the token

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
    "httpOnly": true,
    "secure": null,
    "path": "/",
    "domain": null,
    "validationStrategy": "stormpath"
  }
}
```

#### <a name="http-only"></a> httpOnly

This option determines whether or not this cookie can be read by Javascript code
on the browser.

**NOTE**: Disabling this option is a potential security issue, as your access
tokens could be hijacked via XSS.

<a href="#top">Back to Top</a>


#### <a name="secure"></a> secure

This option determines whether or not our cookies can be set over a non-https
connection.  By default, this option is null, which means we will attempt to
inspect the protocol of the incoming request, and make a decision of what to do
automatically.

Defining this value forces the `secure` mode to be either enabled or disabled
all the time.

**NOTE**: Disabling this option is a potential security issue, as setting
cookies over a non-https connection is ripe for traffic inspection attacks.

<a href="#top">Back to Top</a>


#### <a name="path"></a> path

This option determines what URI paths will be able to read our cookies.  By
default, this is set to the root path (`/`), meaning our cookies will be
available to every URI on the web server.

<a href="#top">Back to Top</a>


#### <a name="domain"></a> domain

This option determines what domain(s), can read this cookie value.  By default,
this is `null`, meaning only the current domain where the cookie is set can use
read this cookie value.

The only time you might want to set this value is if you want to allow
subdomains to access your cookies as well.

<a href="#top">Back to Top</a>


#### <a name="validation-strategy"></a> validationStrategy

Determines how we validate an access token.  There are three strategies:

* **local** - only verify the signature, expiration, and issuer
* **stormpath** - always validate against Stormpath, for additional checks
such as Account and Application status.  This is the default, but the most
consistent (will always be aware of Stormpath resource statuses).
* **random** - 50/50 chance, per request, of **local** or **stormpath** being used.
This balances speed with consistency.

<a href="#top">Back to Top</a>


## <a name="incoming-request-handling"></a> Incoming Request Handling

For any request that comes into the framework, we attempt to validate the access
token, and, if invalid, we'll attempt to extend the session by getting and
setting a new access token (using the refresh token).

If the access token is valid, we'll simply fetch the account object and attach
it to the request context.

If for any reason, the refresh token is invalid, we'll immediately set the
`accessToken` and `refreshToken` cookie values to empty strings, ensuring they
are removed from the browser.

If the developer has declared, "loginRequired" for this route, and if the
requested content type is `json`, we'll return a 401 response -- otherwise if
the content type  is HTML, we should 302 Redirect to the login view.  This flow
ensures that both client side JS frameworks and server side frameworks work as
expected.


<a href="#top">Back to Top</a>


## <a name="me-handling"></a> /me Handling

In situations where a cookie cannot be accessed via Javascript (eg: httpOnly is
true), in order to retrieve the current user's account information, our
libraries should provide an optional route (`/me`), which returns the current
user's account information as a JSON object.

**Example Request**

```
GET /me
```

**Example Response**

```json
{
  "href": "https://api.stormpath.com/v1/accounts/xxx",
  "username": "robert@stormpath.com",
  "email": "robert@stormpath.com",
  "givenName": "Robert",
  "middleName": null,
  "surname": "Damphousse",
  "fullName": "Robert Damphousse",
  "status": "ENABLED",
  "createdAt": "2014-04-07T16:38:44.000Z",
  "modifiedAt": "2014-08-28T22:35:10.000Z",
  "emailVerificationToken": null,
  "customData": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/customData"
  },
  "providerData": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/providerData"
  },
  "directory": {
    "href": "https://api.stormpath.com/v1/directories/xxx"
  },
  "tenant": {
    "href": "https://api.stormpath.com/v1/tenants/xxx"
  },
  "groups": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/groups"
  },
  "applications": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/applications"
  },
  "groupMemberships": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/groupMemberships"
  },
  "apiKeys": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/apiKeys"
  }
}
```

**NOTE**: If a request is made to `/me?`, and query strings used will be passed
into our underlying SDK.  This is useful for things like expanding custom data,
etc.

<a href="#top">Back to Top</a>
