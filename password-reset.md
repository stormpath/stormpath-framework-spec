<a name="#top">Back to Top</a>

# Password Reset


## Table of Contents

* [Options](#Options)
  * [AUTO_LOGIN](#AUTO_LOGIN)
  * [FORGOT_PASSWORD_URL](#FORGOT_PASSWORD_URL)
  * [NEXT_URI](#NEXT_URI)
  * [ERROR_URI](#ERROR_URI)
  * [RESET_PASSWORD_URL](#RESET_PASSWORD_URL)

* [GET Handling, FORGOT_PASSWORD_URL](#GET_FORGOT_PASSWORD_URL)
* [POST Handling, FORGOT_PASSWORD_URL](#POST_FORGOT_PASSWORD_URL)
* [GET Handling, RESET_PASSWORD_URL](#GET_RESET_PASSWORD_URL)
* [POST Handling, RESET_PASSWORD_URL](#POST_RESET_PASSWORD_URL)


## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service password reset of existing user accounts.

If an application's default account store has the password reset workflow
enabled, our library MUST intercept incoming GET requests for the
`FORGOT_PASSWORD_URL` AND `RESET_PASSWORD_URL` and render the appropriate
view, as an HTML Page or Single Page Application.

The `FORGOT_PASSWORD_URL` should render a form which allows the user to request
a password reset token to be sent to their email address.

The `RESET_PASSWORD_URL` should render a form which allows the user to reset
their password, but only after verifying the `sptoken` that is in the URL (they
arrived by clicking on the link that we sent to their email address).

## <a name="Options"></a> Options

This table is a list of all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (e.g. to camel case, or not)? Is not specified here.

| Option                           | Default Value                     |
| -------------------------------- |-----------------------------------|
| AUTO_LOGIN                       | False                             |
| ERORR_URI                        | /forgot?status=INVALID_SP_TOKEN   |
| FORGOT_PASSWORD_URL              | /forgot                           |
| NEXT_URI                         | /login?status=RESET               |
| RESET_PASSWORD_URL               | /reset                            |


#### <a name="AUTO_LOGIN"></a> AUTO_LOGIN

If enabled, will create the `access_token` cookie and redirect the user to the
`NEXT_URI` after they have reset their password.

<a href="#top">Back to Top</a>


#### <a name="RESET_PASSWORD_URL"></a> RESET_PASSWORD_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>


#### <a name="FORGOT_PASSWORD_URL"></a> FORGOT_PASSWORD_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>


#### <a name="ERROR_URI"></a> ERROR_URI

Where to send the user to, if they arrive on the RESET_PASSWORD_URL with an
invalid `spToken`

<a href="#top">Back to Top</a>


#### <a name="NEXT_URI"></a> NEXT_URI

Where to send the user after a successful password reset.

<a href="#top">Back to Top</a>


## <a name="GET_FORGOT_PASSWORD_URL"></a> GET Handling, FORGOT_PASSWORD_URL

This describes how we handle the response for GET requests to the
`FORGOT_PASSWORD_URL`.

The response should be an HTML page or SPA that provides a form with an email
address field, allowing the user to give the email address of the account that
they want to trigger a password reset email for.

The view must have a conditional block for showing an error message, if the user
arrives here with the param `?status=INVALID_SP_TOKEN`

<a href="#top">Back to Top</a>


## <a name="POST_FORGOT_PASSWORD_URL"></a> POST Handling, FORGOT_PASSWORD_URL

The POST request must be of this format:

```
{
  "login": "email or username"
}
```

Regardless of whether or not the login corresponds to an account, we redirect to
the NEXT_URI.  By default, this is our login page which displays a mesage: *"If
the email is  associated with an account, you will receive an email from us
shortly."*

If the request is `Accept: application/json`, the status of the response should
be an HTTP 200 and there should be no body.

<a href="#top">Back to Top</a>


## <a name="GET_RESET_PASSWORD_URL"></a> GET Handling, RESET_PASSWORD_URL

This describes how we handle the response for GET requests to the
`RESET_PASSWORD_URL`, when the user has arrived at our site with an `spToken`
URL parameter.

The request handler must verify the `spToken` with the Stormpath API but should
NOT consume the token as this point.

If the token is valid:

* Render a form for setting a new password on the account

* If the request is `Accept: application/json`, reply with 200 OK and an empty
body

If the token is invalid or expired:

* Redirect to the ERROR_URI

* If the request is `Accept: application/json`, reply with status 400 and a JSON
body of the format `{ error: String  }`

<a href="#top">Back to Top</a>



## <a name="POST_RESET_PASSWORD_URL"></a> POST Handling, RESET_PASSWORD_URL

The POST request must be of this format:

```
{
  "sptoken": "the sent token",
  "password": "new password"
}
```

The token and password should be supplied to the Stormpath API, consuming the token
so that it cannot be used for future requets.

If the operation is successful:

 * And the request is `Accept: text/html`:
  * Redirect to `NEXT_URI`
 * If the request is `Accept: application/json`, the status of the response must
   be an HTTP 200 and there should be no body.

If the operation fails:

* If the request is `Accept: text/html`, re-render the forgot password form with
an error message.

* If the request is `Accept: application/json`, reply with status 400 and a JSON
body of the format `{ error: String  }`

<a href="#top">Back to Top</a>

