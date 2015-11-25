<a name="#top">Back to Top</a>

# Password Reset


## Table of Contents

* [Options](#options)
  * [Forgot Password](#forgot-password)
    * [enabled](#forgot-enabled)
    * [uri](#forgot-uri)
    * [nextUri](#forgot-next-uri)
    * [view](#forgot-view)
  * [Change Password](#change-password)
    * [autoLogin](#change-auto-login)
    * [enabled](#change-enabled)
    * [uri](#change-uri)
    * [errorUri](#change-error-uri)
    * [nextUri](#change-next-uri)
    * [view](#change-view)
* [GET Handling, Forgot Password](#get-forgot-password)
* [POST Handling, Forgot Password](#post-forgot-password)
* [GET Handling, Change Password](#get-change-password)
* [POST Handling, Change Password](#post-change-password)


## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service password reset of existing user accounts.

If an application's default account store has the password reset workflow
enabled, our library MUST intercept incoming GET requests for the
`uri` values presented in this document, and render the appropriate views as
an HTML page.


## <a name="options"></a> Options

These tables list all the options that are required by this feature.  Detailed
descriptions follow.  How the option names are translated into the framework
language (*e.g. to camel case, or not?*) is not specified here.


### <a name="forgot-password"></a> Forgot Password

The table below lists all options for the Forgot Password workflow.  These
options determine how we initialize a password reset for a user.

| Option                           | Default Value                     |
| -------------------------------- |-----------------------------------|
| enabled                          | auto-detected by directory config |
| uri                              | /forgot                           |
| nextUri                          | /login?status=forgot              |
| view                             | forgot                            |


#### <a name="forgot-enabled"></a> enabled

If this feature is turned on, then we will allow users to initialize a password
reset workflow.

**NOTE**: This option will be set to whatever value is specified in the
Directory's configuration.  If the Password Reset workflow is enabled in the
Stormpath Directory -- this option will be automatically enabled.

<a href="#top">Back to Top</a>


#### <a name="forgot-uri"></a> uri

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>


#### <a name="forgot-next-uri"></a> nextUri

This is the URI we'll redirect the user to after a user has successfully
initialized the Password Reset workflow.

<a href="#top">Back to Top</a>


#### <a name="forgot-view"></a> view

This is the name of the view that will be used to render the Password Reset
initialization page.  This page allows a user to enter their account email
address to initialize the Password Reset workflow.

<a href="#top">Back to Top</a>


### <a name="change-password"></a> Change Password

The table below lists all options for the Change Password workflow.  These
options determine how we let a user actually reset their password after they've
clicked the link in their email prompting them to complete the reset process.

| Option                           | Default Value                     |
| -------------------------------- |-----------------------------------|
| autoLogin                        | false                             |
| enabled                          | auto-detected by directory config |
| uri                              | /reset                            |
| errorUri                         | /forgot?status=invalid_sp_token   |
| nextUri                          | /login?status=reset               |
| view                             | reset                             |

<a href="#top">Back to Top</a>


#### <a name="change-auto-login"></a> autoLogin

If this feature is turned on, then once a user has successfully reset his
password, he will be automatically logged into his account and redirected to the
`nextUri` field for the `login` feature.

<a href="#top">Back to Top</a>


#### <a name="change-enabled"></a> enabled

If this feature is turned on, then we will allow users to complete a password
reset workflow.

**NOTE**: This option will be set to whatever value is specified in the
Directory's configuration.  If the Password Reset workflow is enabled in the
Stormpath Directory -- this option will be automatically enabled.

<a href="#top">Back to Top</a>


#### <a name="change-uri"></a> uri

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>


#### <a name="change-error-uri"></a> errorUri

This is the URI we'll redirect a user to if an incoming request is invalid (*eg:
the Stormpath token is invalid.*).

<a href="#top">Back to Top</a>


#### <a name="change-next-uri"></a> nextUri

This is the URI we'll redirect the user to after a user has successfully
reset their account password.

<a href="#top">Back to Top</a>


#### <a name="change-view"></a> view

This is the name of the view that will be used to render the Password Reset
change page.  This page allows a user to enter their new password twice, and
will complete the Password Reset workflow.

<a href="#top">Back to Top</a>


## <a name="get-forgot-password"></a> Get Handling, Forgot Password

This describes how we handle the response for GET requests to the `uri` option
for the Forgot Password workflow.

The response should be an HTML page that provides a form with an email address
field, allowing the user to give the email address of the account that
they want to trigger a password reset email for.

The view must have a conditional block for showing an error message, if the user
arrives here with the parameter `?status=invalid_sp_token`.

<a href="#top">Back to Top</a>


## <a name="post-forgot-password"></a> Post Handling, Forgot Password

This describes how we handle the response for POST requests to the
`uri` option for the Forgot Password workflow.

This handler should accept both `www-form-urlencoded` as well as
`application/json` content.

The POST request must be of this format for form encoded data:

```
login=emailorusername
```

The POST request must be of this format for JSON:

```json
{
  "login": "email or username"
}
```

If the Accept header of the request is `www-form-urlencoded`, then regardless
of whether or not the login corresponds to an account, the user will be
redirected to the `nextUri`.  By default, this will be our login page, which
displays a message that says: *"If the email is associated with an account,
you will receive an email from us shortly."*

If the Accept header of the request is `application/json`, then the status of the
response should be an HTTP 200, and there should be no body.

<a href="#top">Back to Top</a>


## <a name="get-change-password"></a> Get Handling, Change Password

This describes how we handle the response for GET requests to the
`uri` option for the Change Password workflow.

When we receive an incoming request, we should:

- Check to ensure an `spToken` query parameter exists.
- Make an API request to Stormpath to validate the `spToken`.  This should
  **not** consume the token.
- If the `spToken` is invalid, we should redirect the user to the `errorUri`
  option.
- If the `spToken` is valid, we should render a page that contains a form which
  collects the user's new password (*twice*).

<a href="#top">Back to Top</a>


## <a name="post-change-password"></a> Post Handling, Change Password

This describes how we handle the response for POST requests to the
`uri` option for the Change Password workflow.

When we receive an incoming request, we will first ensure that the body of the
request is in the right format.

If the Content Type header is `www-form-urlencoded`, the request should look
like:

```
sptoken=token&password=password
```

If the Content Type header is `application/json`, the request should look like:

```json
{
  "sptoken": "the sent token",
  "password": "new password"
}
```

Once we receive the `sptoken` and `password` values, we should consume the token
using the Stormpath API, to ensure it can no longer be used for future requests.

If the operation is successful and the Accept header is set to `text/html`, we
should then redirect the user to the `nextUri` option for the Change Password
workflow.

If the operation is successful and the Accept header is set to
`application/json`, we should return an HTTP 200 with no body.

If the operation fails and the Accept header is set to `text/html`, we should
redirect the user to the `errorUri` option for the Change Password workflow.

If the operation fails and the Accept header is set to `application/json`, we
should return an HTTP 400 with a JSON body that contains
`{ "error": "message" }`.

<a href="#top">Back to Top</a>
