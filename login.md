<a name="#top">Back to Top</a>

# Account Login


## Table of Contents

* [Options](#Options)
  * [AUTO_LOGIN](#AUTO_LOGIN)
  * [ENABLED](#ENABLED)
  * [REDIRECT_URL](#REDIRECT_URL)
  * [LOGIN_URL](#LOGIN_URL)
* [POST Body Format](#POST_Body_Format)
* [POST Error Handling](#POST_Error_Handling)
* [POST Response Handling](#POST_Response_Handling)


## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service login of user accounts.

If enabled via the `ENABLED` option, our library MUST intercept
incoming requests for the `URI` and either render a login form (GET) or
handle a POST request from the login form.

GET requests should serve an HTML Page OR Single Page Application, in either
case the user should be presented with a login form.

The form MUST:

* Require an email address / username AND password OR
* At least one form of social login
* Render social login buttons, if social provider account stores exist (see [social.md][])


## <a name="Options"></a> Options

This table is a list of all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (e.g. to camel case, or not)? Is not specified here.

| Option                           | Default Value       |
| -------------------------------- |---------------------|
| AUTO_REDIRECT                    | True                |
| ENABLED                          | True                |
| REDIRECT_URL                     | /                   |
| LOGIN_URL                        | /login              |


#### <a name="AUTO_REDIRECT"></a> AUTO_REDIRECT

If enabled, and a user already has a valid session, instead of re-rendering the
login page we will redirect this user to the URL specified by `REDIRECT_URL`.

If disabled, we won't redirect the user anywhere and will simply re-render the
login page.  However -- in this case we will *also* destroy any existing user
sessions.  This ensures that odd edge cases won't come up wherein a user is
viewing a login page but can see their account information in some place (like a
menu bar).

<a href="#top">Back to Top</a>


#### <a name="ENABLED"></a> ENABLED

If `True` this feature will be enabled and our library will intercept requests
at the [URI](#URI).  When the application server starts, we will
query the user's Stormpath Application to discover what Account Store Mappings
are available.  We will then pre-load these so that the login page displays all
available forms of login (*this might include username / email and password
login, Facebook Login, Google Login, etc.*).

If `False` this feature is disabled and the base framework will be responsible
for the [URI](#URI), likely resulting in a 404 Not Found error.

**NOTE**: If this feature is enabled, and no Account Stores are mapped to this
Application -- we will throw an error during initialization since there are no
possible ways for a user to authenticate.


<a href="#top">Back to Top</a>


#### <a name="REDIRECT_URL"></a> REDIRECT_URL

Where to send the user after successful login, if
[ENABLED](#ENABLED) is `True`

<a href="#top">Back to Top</a>


#### <a name="URI"></a> URI

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>

## <a name="POST_Body_Format"></a> POST Body Format

This endpoint accepts password based login, and social login.  For page-rendered
flows the form will be submitted as `application/x-www-form-urlencoded`.  For
front-end applications the form will be submitted as `application/json`.

**Password-based login**

In this situation, we render a form when the login page is requested.  The form
must submit the following fields:

```x-www-form-urlencoded
login=robert@stormpath.com
password=mypassword
```

* The `login` field can be either a username or email.

* If either field is omitted, an error will be raised and the page will be
re-rendered.

**Social login**

In this situation, the front-end client has obtained an access code or authorization
code from the provider.  The front-end client is now submitting this information,
along with our providerId for the provider.

Page-based social login flows are described in [social.md][] and
handled by different endpoints

```json
{
    "providerId": "google",
    "accessToken": "xxx",
    "code": "xxx"
}
```

* Only one of `accessToken` or `code` will be provided, both are listed
  for example purposes only.

<a href="#top">Back to Top</a>


##  <a name="POST_Error_Handling"></a> POST Error Handling

**For HTML responses:**

For any errors, the response should be a 200 OK and the form should be
re-rendered with a UX that indicates which field is in error and what can be
done to fix the problem.

**For JSON responses:**

Send a 400 JSON response where the body is of the format
`{ error: 'user friendly error message' }`

## <a name="POST_Response_Handling"></a> POST Response Handling

This describes how we handle the response, after an account has been
successfully authenticated.

**For HTML responses:**

If the newly authenticated account's status is ENABLED then we'll issue a 302
redirect to the REDIRECT_URL and create a new user session.

If the newly authenticated account's status is UNVERIFIED, then we'll render a
view which tells the user to check their email for a verification link.  This
view will also include a link to resend the verification email just incase the
user didn't receive it originally.

If the newly authenticate account's status is DISABLED, then we'll render a view
which tells the user their account has been disabled, and they need to contact
the site administrator for help.

**For JSON responses:**

If the account is retrieved, send a 200 JSON body response, where the body is
the account object.


<a href="#top">Back to Top</a>


## <a name="GET_Request_Handling"></a> GET Request Handling

This describes how we render the login page after receiving a GET request.

If the request type is HTML, we should render a login page with a form that
accepts either a username or email address and a password.  It should render
the login buttons for social providders, if configured (see [social.md][]).

If the request type is JSON, send 405.

If a `status` parameter is specified in the query string, and the status value
is set to `verified`, then we should display a success message above the login
form which says that this account was successfully verified, and that the user
can log into their account below.

<a href="#top">Back to Top</a>

[social.md]: social.md
