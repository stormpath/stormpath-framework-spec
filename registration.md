<a name="#top">Back to Top</a>

# Account Registration


## Table of Contents

* [Options](#Options)
* [POST Body Format](#POST_Body_Format)
* [POST Error Handling](#POST_Error_Handling)
* [POST Response Handling](#POST_Response_Handling)

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service registration of user accounts.

If enabled via the ENABLED option, our library MUST intercept
incoming requests for the `uri` and either render a registration
form (GET) or handle a POST request from the registration form.

GET requests should serve an HTML Page OR Single Page Application, in either
case the user should be presented with a registration form.

The form MUST:

* Require a first name
* Require a last name
* Require an email address
* Require a password

The form MAY:

* Have other fields:
 * Username
 * Middle Name
 * Password confirmation field

If the following account properties are not specified by the user in the form
POST, our library MUST set the value to 'UNKNOWN' when we create the account:

 * Given Name
 * Surname

## <a name="Options"></a> Options

This is the set of default configuration for the registration feature, it
defines the default fields we support and which ones we want to show by
default.  If a field is `enabled`, it shoud be shown on the registration form.
If it is `required`, the form must show an erorr if the value is not supplied.

```yaml
stormpath:
  web:
  register:
    enabled: false
    uri: "/register"
    nextUri: "/"
    autoLogin: false
    fields:
      givenName:
        enabled: true
        name: "givenName"
        placeholder: "First Name"
        required: true
        type: "text"
      middleName:
        enabled: false
        name: "middleName"
        placeholder: "Middle Name"
        required: true
        type: "text"
      surname:
        enabled: true
        name: "surname"
        placeholder: "Last Name"
        required: true
        type: "text"
      username:
        enabled: false
        name: "username"
        placeholder: "Username"
        required: true
        type: "text"
      email:
        enabled: true
        name: "email"
        placeholder: "Email"
        required: true
        type: "email"
      password:
        enabled: true
        name: "password"
        placeholder: "Password"
        required: true
        type: "password"
      confirmPassword:
        enabled: false
        name: "confirmPassword"
        placeholder: "Conrim Password"
        required: true
        type: "password"
    fieldOrder:
      - "username"
      - "givenName"
      - "middleName"
      - "surname"
      - "email"
      - "password"
      - "confirmPassword"
    view: "register"
```
#### <a name="autoLogin"></a> autoLogin

If enabled, will create the `access_token` cookie and redirect the user to the
nextUri if they have successfully registered.  See the
[POST Response Handling](#POST_Response_Handling) section for more details.
Works in conjunction with our Email Verification feature.

<a href="#top">Back to Top</a>



#### <a name="nextUri"></a> nextUri

Where to send the user after successful registration, if
[autoLogin](#autoLogin) is `True`

<a href="#top">Back to Top</a>


#### <a name="`uri`"></a> `uri`

This is the URI portion of an entire URL that our library will attach an
interceptor to for GET and POST requests.

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

If those fields are omitted, it is an error.  If any of the required fields are
omitted, that is also an error.

The POST body may contain custom data, which should be passed along to the
Stormpath API when creating the account.

```json
{
    "email": "robert@stormpath.com",
    "password": "d",
    "customValue": "custom value on object root needs to go in custom data, too",
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
  * The newly created account's status is ENABLED and [autoLogin](#autoLogin)
    is:
    * `True`: issue a 302 Redirect to the nextUri and create set
      `access_token` cookie on the client
    * `False`: inform the user that their account has been created and they may
      now login
  * The newly created account status is UNVERIFIED, then render a view which
  tells the user to check their email for a verification link.  This view should
  containa  link to the VERIFY view, reading "didn't get the email? click here
  to re-send the message."

<a href="#top">Back to Top</a>
