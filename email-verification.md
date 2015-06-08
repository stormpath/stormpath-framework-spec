<a name="#top">Back to Top</a>

# Email Verification


## Table of Contents

* [Options](#Options)
  * [AUTO_LOGIN](#AUTO_LOGIN)
  * [EMAIL_VERIFICATION_URL](#EMAIL_VERIFICATION_URL)
  * [REDIRECT_URL](#REDIRECT_URL)
* [GET Error Handling](#GET_Error_Handling)
* [GET Response Handling](#GET_Response_Handling)
* [POST Error Handling](#POST_Error_Handling)
* [POST Response Handling](#POST_Response_Handling)


## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service verification of newly registered user accounts.

If an application's default account store has the email verification workflow
enabled, our library MUST intercept incoming GET requests for the
`EMAIL_VERIFICATION_URL` and either render an account activation view or an
error view.

GET requests should serve an HTML Page OR Single Page Application, in either
case the user should be presented with the appropriate view.


## <a name="Options"></a> Options

This table is a list of all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (e.g. to camel case, or not)? Is not specified here.

| Option                           | Default Value       |
| -------------------------------- |---------------------|
| AUTO_LOGIN                       | True                |
| EMAIL_VERIFICATION_URL           | /verify             |
| REDIRECT_URL                     | /                   |


#### <a name="AUTO_LOGIN"></a> AUTO_LOGIN

If enabled, will create the `access_token` cookie and redirect the user to the
`REDIRECT_URL` if they have successfully verified their account.

<a href="#top">Back to Top</a>


#### <a name="EMAIL_VERIFICATION_URL"></a> EMAIL_VERIFICATION_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET requests.

<a href="#top">Back to Top</a>


#### <a name="REDIRECT_URL"></a> REDIRECT_URL

Where to send the user after successful verification.  This is where we'll send
the user after showing them a confirmation message for 5 seconds.

<a href="#top">Back to Top</a>


## <a name="GET_Error_Handling"></a> GET Error Handling

This describes how we handle the response for the `EMAIL_VERIFICATION_URL`
controller after the user has arrived at our site with an `spToken` URL
parameter.

If the `spToken` is invalid, we'll render a failure view to the user which
contains a link to another page where a user can optionally resend their
verification email to themselves.

If the request is `Accept: application/json`, the response should be a JSON
object of the format `{ error: String  }` where String is a user-friendly
message and the status of the response should be 400.

<a href="#top">Back to Top</a>


## <a name="GET_Response_Handling"></a> GET Response Handling

This describes how we handle the response for the `EMAIL_VERIFICATION_URL`
controller, after the user has arrived at our site with an `spToken` URL
parameter.

If the `spToken` is valid, we'll render the confirmation view to the user and
consume the `spToken` with our underlying API -- then eventually redirect the
user to the `REDIECT_URL` setting with a new user session (if `AUTO_LOGIN` is
True).

If the request is `Accept: application/json`, the status of the response should
be an HTTP 200 and there should be no body.

If not `spToken` param is present, this should render an HTML page with a form
that allows the user to enter their account email address and will attempt to
resend the verification email to the user.

<a href="#top">Back to Top</a>


## <a name="POST_Error_Handling"></a> POST Error Handling

Regardless of whether or not the email address the user entered was successful,
we should render a success message along the lines of *"If the email address you
entered was associated with an account, you will receive an email from us
shortly."*

If the request is `Accept: application/json`, the status of the response should
be an HTTP 200 and there should be no body.

<a href="#top">Back to Top</a>


## <a name="POST_Response_Handling"></a> POST Response Handling

This describes how we handle the POST response for the `EMAIL_VERIFICATION_URL`
controller, after the user has submitted form data.

Regardless of whether or not the email address the user entered was successful,
we should render a success message along the lines of *"If the email address you
entered was associated with an account, you will receive an email from us
shortly."*

If the request is `Accept: application/json`, the status of the response should
be an HTTP 200 and there should be no body.

<a href="#top">Back to Top</a>
