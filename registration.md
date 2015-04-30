<a name="#top">Back to Top</a>

# Account Registration

* [Default Options](#Default_Options)
* [Option Descriptions](#Option_Descriptions)
  * [AUTO_LOGIN](#AUTO_LOGIN)
  * [POST_REGISTRATION_URL](#POST_REGISTRATION_URL)
* [POST Body Format](#POST_Body_Format)
* [POST Error Handling](#POST_Error_Handling)
* [POST Response Handling](#POST_Response_Handling)

This document describes the endpoints and logic that must exist in order to
facilitate self-service registration of user accounts.

If enabled via the ENABLE_REGISTRATION option, the framework MUST intercept
incoming requests for the POST_REGISTRATION_URL and either render a registration
form or handle a POST request from the registration form.

## <a name="Default_Options"></a> Default Options

This table is a list of all the options that are required by this feature.
Detailed descriptions follow.  How the option names are translated into the
framework language (e.g. to camel case, or not)? Is not specified here.

| Option                           | Default Value |
| -------------------------------- |---------------|
| AUTO_LOGIN                       | false         |
| ENABLE_REGISTRATION              | true          |
| POST_REGISTRATION_REDIRECT       | /             |
| POST_REGISTRATION_URL            | /register     |
| REQUIRE_GIVEN_NAME               | true          |
| REQUIRE_PASSWORD_CONFIRMATION    | false         |
| REQUIRE_SURNAME                  | true          |

## <a name="Option_Descriptions"></a> Option Descriptions

#### <a name="AUTO_LOGIN"></a> AUTO_LOGIN

If enabled AND the the request is `Accept: text/html` AND an account is
successfully created AND the account status is ENABLED, then issue a 302
Redirect to the POST_REGISTRATION_URL

<a href="#top">Back to Top</a>

#### <a name="POST_REGISTRATION_URL"></a> POST_REGISTRATION_URL

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

The POST body may contain custom data, which should be passed along to the API
when creating the account

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

For any errors indicated below, the following should happen:

* If the request is `Accept: text/html`, the response should be 200 OK and the
form should be re-rendered with a UX that indicates which field is in error and
what can be done to fix the problem.

* If the request is `Accept: application/json`, the response should be a JSON
object of the format `{ error: String }` where String is a user-friendly message
and the status of the response should be 400.

## <a name="POST_Response_Handling"></a> POST Response Handling (Account was created)

If the request is `Accept: application/json`, the response should always be
status 200 and the body should be a JSON body which is the account object that
was created, but DO NOT expand any resource on the account object.  We do not
want to leak any information that may have been associated with this account
(i.e. custom data)

<a href="#top">Back to Top</a>
