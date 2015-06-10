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
* [POST Body Format](#POST_Body_Format)
* [POST Error Handling](#POST_Error_Handling)
* [POST Response Handling](#POST_Response_Handling)


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

| Option                           | Default Value       |
| -------------------------------- |---------------------|
| AUTO_REDIRECT                    | True                |
| ENABLE_LOGIN                     | False               |
| REDIRECT_URL                     | /                   |
| LOGIN_URL                        | /login              |
| GOOGLE_CALLBACK_URL              | /callbacks/google   |
| FACEBOOK_CALLBACK_URL            | /callbacks/facebook |
| GITHUB_CALLBACK_URL              | /callbacks/github   |
| LINKEDIN_CALLBACK_URL            | /callbacks/linkedin |


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


#### <a name="REDIRECT_URL"></a> REDIRECT_URL

Where to send the user after successful login, if
[ENABLE_LOGIN](#ENABLE_LOGIN) is `True`

<a href="#top">Back to Top</a>


#### <a name="LOGIN_URL"></a> LOGIN_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>


#### <a name="GOOGLE_CALLBACK_URL"></a> GOOGLE_CALLBACK_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to GET requests in order to handle the Google Login flow.

This interceptor will read in the access token from the query parameters, create
or update the account in Stormpath, create a session for the user, and finally
redirect the user to the `REDIRECT_URL` url.

<a href="#top">Back to Top</a>


#### <a name="FACEBOOK_CALLBACK_URL"></a> FACEBOOK_CALLBACK_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to GET requests in order to handle the Facebook Login flow.

This interceptor will read in the access token from the query parameters, create
or update the account in Stormpath, create a session for the user, and finally
redirect the user to the `REDIRECT_URL` url.

<a href="#top">Back to Top</a>


#### <a name="GITHUB_CALLBACK_URL"></a> GITHUB_CALLBACK_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to GET requests in order to handle the Github Login flow.

This interceptor will read in the access token from the query parameters, create
or update the account in Stormpath, create a session for the user, and finally
redirect the user to the `REDIRECT_URL` url.

<a href="#top">Back to Top</a>


#### <a name="LINKEDIN_CALLBACK_URL"></a> LINKEDIN_CALLBACK_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to GET requests in order to handle the LinkedIn Login flow.

This interceptor will read in the access token from the query parameters, create
or update the account in Stormpath, create a session for the user, and finally
redirect the user to the `REDIRECT_URL` url.

<a href="#top">Back to Top</a>


## <a name="POST_Body_Format"></a> POST Body Format

The content type of the POST must be `application/x-www-form-urlencoded`.

This endpoint only accepts logins with either a username or password specified,
as social login operates differently.

Here is an example request.

```json
{
    "login": "robert@stormpath.com",
    "password": "d"
}
```

The `login` field can be either a username or email.

If either field is omitted, an error will be raised and the page will be
re-rendered.

<a href="#top">Back to Top</a>


##  <a name="POST_Error_Handling"></a> POST Error Handling

For any errors, the response should be a 200 OK and the form should be
re-rendered with a UX that indicates which field is in error and what can be
done to fix the problem.


## <a name="POST_Response_Handling"></a> POST Response Handling

This describes how we handle the response, after an account has been
successfully authenticated.

If the newly authenticated account's status is ENABLED then we'll issue a 302
redirect to the REDIRECT_URL and create a new user session.

If the newly authenticated account's status is UNVERIFIED, then we'll render a
view which tells the user to check their email for a verification link.  This
view will also include a link to resend the verification email just incase the
user didn't receive it originally.

If the newly authenticate account's status is DISABLED, then we'll render a view
which tells the user their account has been disabled, and they need to contact
the site administrator for help.


<a href="#top">Back to Top</a>
