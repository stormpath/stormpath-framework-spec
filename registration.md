<a name="#top">Back to Top</a>

# Account Registration

This document describes the endpoints and logic that must exist in order to
facilitate self-service registration of user accounts.

If enabled via the ENABLE_REGISTRATION option, the framework MUST intercept
incoming requests for the REGISTRATION_URL and either render a registration
form or handle a POST request from the registration form.

## Table of Contents

* [Default Options](#Default_Options)
* [Option Descriptions](#Option_Descriptions)
  * [AUTO_LOGIN](#AUTO_LOGIN)
  * [REGISTRATION_URL](#REGISTRATION_URL)
* [POST Body Format](#POST_Body_Format)
* [POST Error Handling](#POST_Error_Handling)
* [POST Response Handling](#POST_Response_Handling)

## <a name="Default_Options"></a> Default Options

This table is a list of all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (e.g. to camel case, or not)? Is not specified here.

| Option                           | Default Value |
| -------------------------------- |---------------|
| AUTO_LOGIN                       | false         |
| ENABLE_REGISTRATION              | true          |
| POST_REGISTRATION_REDIRECT_URL   | /             |
| REGISTRATION_URL                 | /register     |
| REQUIRE_GIVEN_NAME               | true          |
| REQUIRE_PASSWORD_CONFIRMATION    | false         |
| REQUIRE_SURNAME                  | true          |

## <a name="Option_Descriptions"></a> Option Descriptions

#### <a name="AUTO_LOGIN"></a> AUTO_LOGIN

If enabled, will create the `access_token` cookie and redirect the user to the
POST_REGISTRATION_REDIRECT_URL if they have successfully registered.  See the
[POST Response Handling](#POST_Response_Handling) section for more details.
Works in conjunction with our Email Verification feature.

<a href="#top">Back to Top</a>

#### <a name="REGISTRATION_URL"></a> REGISTRATION_URL

This is the URI portion of an entire URL that the framework will handle GET and
POST requests for.

GET requests should serve an HTML Page OR Single Page Application, in either
case the user should be presented with a registration form.

The form MUST:

* It must require an email address
* It must require a password

The form MAY:

* Have other fields, as controlled by these options:
 * REQUIRE_SURNAME
 * REQUIRE_GIVEN_NAME
 * REQUIRE_PASSWORD_CONFIRMATION

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

If any of the options REQUIRE_GIVEN_NAME, REQUIRE_PASSWORD_CONFIRMATION,
REQUIRE_SURNAME are enabled, it should be considered an error if the relevant
fields are missing.

The POST body may contain custom data, which should be passed along to the
Stormpath API when creating the account

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

* If the request is `Accept: application/json`, the response should always be
status 200 and the body should be a JSON body which is the account object that
was created, but DO NOT expand any resource on the account object.  We do not
want to leak any information that may have been associated with this account
(i.e. custom data)

* If the request is `Accept: text/html`, and..
  * The newly created account's status is ENABLED and [AUTO_LOGIN](#AUTO_LOGIN)
    is:
    * `True`: issue a 302 Redirect to the REGISTRATION_URL and create set
      `access_token` cookie on the client
    * `False`: inform the user that their account has been created and they may
      now login
  * The newly created account status is UNVERIFIED, then render a view which
  tells the user to check their email for a verification link

<a href="#top">Back to Top</a>
