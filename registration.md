<a name="#top">Back to Top</a>

# Account Registration


## Table of Contents

* [Options](#Options)
  * [AUTO_LOGIN](#AUTO_LOGIN)
  * [ENABLE_REGISTRATION](#ENABLE_REGISTRATION)
  * [REDIRECT_URL](#REDIRECT_URL)
  * [REGISTRATION_URL](#REGISTRATION_URL)
* [POST Body Format](#POST_Body_Format)
* [POST Error Handling](#POST_Error_Handling)
* [POST Response Handling](#POST_Response_Handling)

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service registration of user accounts.

If enabled via the ENABLE_REGISTRATION option, our library MUST intercept
incoming requests for the REGISTRATION_URL and either render a registration
form (GET) or handle a POST request from the registration form.

GET requests should serve an HTML Page OR Single Page Application, in either
case the user should be presented with a registration form.

The form MUST:

* Require an email address
* Require a password

The form MAY:

* Have other fields:
 * Username
 * Given Name
 * Middle Name
 * Surname
 * Password confirmation field

If the following account properties are not specified by the user in the form
POST, our library MUST set the value to 'UNKNOWN' when we create the account:

 * Given Name
 * Surname

## <a name="Options"></a> Options

This table is a list of all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (e.g. to camel case, or not)? Is not specified here.

| Option                           | Default Value |
| -------------------------------- |---------------|
| AUTO_LOGIN                       | True          |
| ENABLE_GIVEN_NAME                | False         |
| ENABLE_MIDDLE_NAME               | False         |
| ENABLE_PASSWORD_CONFIRMATION     | False         |
| ENABLE_REGISTRATION              | False         |
| ENABLE_SURNAME                   | False         |
| ENABLE_USERNAME                  | False         |
| REDIRECT_URL                     | /             |
| REGISTRATION_URL                 | /register     |
| REQUIRE_GIVEN_NAME               | False         |
| REQUIRE_MIDDLE_NAME              | False         |
| REQUIRE_SURNAME                  | False         |
| REQUIRE_USERNAME                 | False         |

#### <a name="AUTO_LOGIN"></a> AUTO_LOGIN

If enabled, will create the `access_token` cookie and redirect the user to the
REDIRECT_URL if they have successfully registered.  See the
[POST Response Handling](#POST_Response_Handling) section for more details.
Works in conjunction with our Email Verification feature.

<a href="#top">Back to Top</a>




#### <a name="ENABLE_GIVEN_NAME"></a> ENABLE_GIVEN_NAME

If enabled, expose a field on the form for entering the user's first name.

<a href="#top">Back to Top</a>




#### <a name="ENABLE_MIDDLE_NAME"></a> ENABLE_MIDDLE_NAME

If enabled, expose a field on the form for entering the user's middle name.

<a href="#top">Back to Top</a>




#### <a name="ENABLE_REGISTRATION"></a> ENABLE_REGISTRATION

If `True` this feature will be enabled and our library will intercept requests
at the [REGISTRATION_URL](#REGISTRATION_URL)

If `False` this feature is disabled and the base framework will be responsible
for the [REGISTRATION_URL](#REGISTRATION_URL), likely resulting in a 404
Not Found error.

<a href="#top">Back to Top</a>




#### <a name="ENABLE_SURNAME"></a> ENABLE_SURNAME

If enabled, expose a field on the form for entering the user's last name.

<a href="#top">Back to Top</a>




#### <a name="ENABLE_USERNAME"></a> ENABLE_USERNAME

If enabled, expose a field on the form for entering a username that is separate
from the email address.

<a href="#top">Back to Top</a>




#### <a name="REDIRECT_URL"></a> REDIRECT_URL

Where to send the user after successful registration, if
[AUTO_LOGIN](#AUTO_LOGIN) is `True`

<a href="#top">Back to Top</a>




#### <a name="REGISTRATION_URL"></a> REGISTRATION_URL

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

<a href="#top">Back to Top</a>




#### <a name="REQUIRE_GIVEN_NAME"></a> REQUIRE_GIVEN_NAME

If enabled, the user must specify their first name.  Even if ENABLE_MIDDLE_NAME
is False, the field should still be shown if this property is True

<a href="#top">Back to Top</a>




#### <a name="REQUIRE_MIDDLE_NAME"></a> REQUIRE_MIDDLE_NAME

If enabled, the user must specify their middle name.  Even if ENABLE_MIDDLE_NAME
is False, the field should still be shown if this property is True

<a href="#top">Back to Top</a>




#### <a name="REQUIRE_SURNAME"></a> REQUIRE_SURNAME

If enabled, the user must specify their last name.  Even if ENABLE_SURNAME
is False, the field should still be shown if this property is True

<a href="#top">Back to Top</a>



#### <a name="REQUIRE_USERNAME"></a> REQUIRE_USERNAME

If enabled, the user must specify a username that is separate from the email
address.  Even if ENABLE_USERNAME is False, the field should still be shown if
this property is True

<a href="#top">Back to Top</a>




## <a name="POST_Body_Format"></a> POST Body Format

The content type of the POST may be `application/x-www-form-urlencoded` or
`application/json`, the framework should be configured to accept both.

In this section I will use JSON as the example.


Every POST body MUST contain, at a minimum, these fields:

```json
{
    "email": "robert@stormpath.com",
    "password": "d"
}
```

If those fields are omitted, it is an error.

If any of the options are enabled:

 * ENABLE_PASSWORD_CONFIRMATION
 * REQUIRE_GIVEN_NAME
 * REQUIRE_MIDDLE_NAME
 * REQUIRE_SURNAME
 * REQUIRE_USERNAME

It should be considered an error if the relevant fields are missing.

The POST body may contain custom data, which should be passed along to the
Stormpath API when creating the account.

```json
{
    "email": "robert@stormpath.com",
    "password": "d",
    "customData": {
        "hello": "world"
    }
}
```

<a href="#top">Back to Top</a>

##  <a name="POST_Error_Handling"></a> POST Error Handling

For any errors indicated in this document, the following should happen:

* If the request is `Accept: text/html`, the response should be 200 OK and the
form should be re-rendered with a UX that indicates which field is in error and
what can be done to fix the problem.

* If the request is `Accept: application/json`, the response should be a JSON
object of the format `{ error: String }` where String is a user-friendly message
and the status of the response should be 400.

## <a name="POST_Response_Handling"></a> POST Response Handling

This describes how we handle the response, after an account has been
successfully created.

* If the request is `Accept: application/json`, the response should be status
  200 with a JSON body.  The body should be the account object that was returned
  from the account creation call.
```

* If the request is `Accept: text/html`, and..
  * The newly created account's status is ENABLED and [AUTO_LOGIN](#AUTO_LOGIN)
    is:
    * `True`: issue a 302 Redirect to the REDIRECT_URL and create set
      `access_token` cookie on the client
    * `False`: inform the user that their account has been created and they may
      now login
  * The newly created account status is UNVERIFIED, then render a view which
  tells the user to check their email for a verification link.  This view should
  containa  link to the VERIFY view, reading "didn't get the email? click here
  to re-send the message."

<a href="#top">Back to Top</a>
