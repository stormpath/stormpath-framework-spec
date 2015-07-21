<a name="#top">Back to Top</a>

# Email Verification


## Table of Contents

* [Options](#Options)
  * [autoLogin](#autoLogin)
  * [uri](#uri)
  * [nextUri](#nextUri)
* [GET Error Handling](#GET_ERROR_HANDLING)
* [GET Response Handling](#GET_RESPONSE_HANDLING)
* [POST Error Handling](#POST_ERROR_HANDLING)
* [POST Response Handling](#POST_RESPONSE_HANDLING)


## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service verification of newly registered user accounts.

If an application's default account store has the email verification workflow
enabled, our library MUST intercept incoming GET requests for the
`uri` and either render an account activation view or an
error view.

GET requests should serve an HTML Page OR Single Page Application, in either
case the user should be presented with the appropriate view.


## <a name="Options"></a> Options

This JSON blob lists all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (*e.g. to camel case, or not*)?  Is not specified here.

```json
{
  "verifyEmail": {
    "autoLogin": false,
    "uri": "/verify",
    "nextUri": "/"
  }
}
```


### <a name="autoLogin"></a> autoLogin

If enabled, will create the `access_token` cookie and redirect the user to the
`nextUri?status=verified` if they have successfully verified their account.

<a href="#top">Back to Top</a>


### <a name="uri"></a> uri

This is URI is what we'll attach an interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>


### <a name="nextUri"></a> nextUri

Where to send the user after successful verification.

<a href="#top">Back to Top</a>


## <a name="GET_ERROR_HANDLING"></a> GET Error Handling

This describes how we handle the response for the `uri` controller after the
user has arrived at our site with an `spToken` URL parameter.

If the `spToken` query parameter is invalid, we'll render a failure view to the
user which contains a form that:

- Displays an error message, and
- Allows a user to enter their email address to resend the verification email to
  themselves.

If the request is `Accept: application/json`, the response should be a JSON
object of the format `{ error: String  }` where String is a user-friendly
message and the status of the response should be 400.

<a href="#top">Back to Top</a>


## <a name="GET_RESPONSE_HANDLING"></a> GET Response Handling

This describes how we handle the response for the `uri` controller, after the
user has arrived at our site with an `spToken` query parameter.

If `spToken` is valid, we'll render the confirmation view to the user and
consume the `spToken` with our underlying API -- then redirect the
user to the `nextUri?status=verified` setting with a new user session (if
`autoLogin` is `true`).  If `autoLogin` is `false`, notify the user that their
account is verified and provide a link to the login page.

If the request is `Accept: application/json`, the status of the response should
be an HTTP 200 and there should be no body.

<a href="#top">Back to Top</a>


## <a name="POST_ERROR_HANDLING"></a> POST Error Handling

Regardless of whether or not the email address the user entered was successful,
we should render a success message along the lines of *"If the email address you
entered was associated with an account, you will receive an email from us
shortly."*

If the request is `Accept: application/json`, the status of the response should
be an HTTP 200 and there should be no body.

<a href="#top">Back to Top</a>


## <a name="POST_RESPONSE_HANDLING"></a> POST Response Handling

This describes how we handle the POST response for the `uri`
controller, after the user has submitted form data.

Regardless of whether or not the email address the user entered was successful,
we should render a success message along the lines of *"If the email address you
entered was associated with an account, you will receive an email from us
shortly."*

If the request is `Accept: application/json`, the status of the response should
be an HTTP 200 and there should be no body.

<a href="#top">Back to Top</a>
