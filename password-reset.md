<a name="#top">Back to Top</a>

# Password Reset

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service password reset of existing user accounts.

This feature has two endpoints:

  * The `/forgot` endpoint, which is used for requesting a password reset email.

  * The `/change` endpoint, which is used for setting a new password and is
    linked to from the password reset email.  This is where the user arrives
    with the `sptoken` that was sent in the email.


## Forgot Password Endpoint (`/forgot`)

This is the endpoint that a user will use if they want to request a password
reset email.

If the default account store of the stormapth application has the password reset
workflow enabled, and `stormpath.web.forgotPassword.enabled` is not set to
`false`, our library MUST inspect incoming requests at
`stormpath.web.forgotPassword.uri` and follow the request handling procedure
that is defined below, to determine if a response is required.

### Request Handling

#### GET Requests

If the request prefers `text/html`, render the forgot-password form that is
defined by `stormpath.web.forgotPassword.view`.

There is no JSON response for the GET verb of this endpoint.

#### POST Requests

This endpoint accepts a post from the password reset request form, and the only
field in this form is the `email` field.  The endpoint should parse the POST body
content as `application/json` or `application/x-www-form-urlencoded`.

The format of the request is (JSON example):

```javascript
{
  "email": "foo@bar.com"
}
```

Regardless of whether or not the email address is associated with a user
account, we must do the following:

* If the request prefers `application/html`, redirect the user to
  `stormpath.web.forgotPassword.nextUri`

* If the request prefers `application/json`, respond with `200 OK` and no body.

### <a name="options"></a> Options

```yaml
stormpath:
  web:
    forgotPassword:
      enabled: null
      uri: "/forgot"
      view: "forgot-password"
      nextUri: "/login?status=forgot"
```

#### enabled

Default: `null`

Unless explicitly set to `false`, this endpoint must be automatically enabled if
the default account store for the defined Stormpath application has the password
reset workflow enabled.

<a href="#top">Back to Top</a>


#### uri

Default: `/forgot`

The URI that we'll attach an interceptor to for requests, if `enabled` is
`true`.

<a href="#top">Back to Top</a>


#### nextUri

Default: `/login?status=forgot`

This is the URI we'll redirect the user to after a user has successfully
initialized the Password Reset workflow by submitting an email address.

<a href="#top">Back to Top</a>


#### view

Default: `forgot-password`

A string key which identifies the view template that should be used.

<a href="#top">Back to Top</a>


## Change Password Endpoint (`/change`)

This is the endpoint that a user will use if they have received a password
reset email and have clicked on the link in the email.  The link will point to
this endpoint, and contain the `sptoken` query parameter.  For HTML requests, we
verify the `sptoken` and then render a form that the user will use to submit a
new password.



### Request Handling

#### GET Requests

* If there is a `?sptoken` query parameter in the URL:

  * Make an API request to Stormpath to validate the `sptoken`, but **do not**
    consume the token yet.

  * If the `sptoken` is invalid, and the request prefers:

    * `text/html`, redirect the user to `stormpath.web.changePassword.errorUri`.

    * `application/json`, respond with the JSON error from the API, according to
      the [Error Handling][] specification.

  * If the `sptoken` is valid, and the request prefers:

    * `text/html`, render the change password view that is defined by
      `stormpath.web.changePassword.view`.

    * `application/json`, reply with 200 OK.

* If there isn't a `?sptoken` query parameter in the URL, and the request
  prefers:

  * `text/html`, redirect the user to `stormpath.web.forgotPassword.uri`.

  * `application/json`, send this error:

    ```javascript
    {
      status: 400,
      message: 'sptoken parameter not provided.'
    }
    ```

#### POST Handling

This endpoint accepts a post from the password change form.  The endpoint should
parse the post body as `application/json` or `application/x-www-form-urlencoded`.

`application/x-www-form-urlencoded` requests look like:

```
POST /change?sptoken=the_sent_token

password=new%20password&passwordAgain=new%20password
```

`application/json` requests look like:

```javascript
POST /change

{
  "sptoken": "the sent token",
  "password": "new password"
  "passwordAgain": "new password"
}
```

Before making a network request to Stormpath, the `password` and `passwordAgain` parameters should be checked to make sure they match.

* If the check fails, re-render the change password form (see **Error Responses** below).

* If the check passes, the Stormpath API should be invoked to consume the token and reset the password.

**Success Responses**

If the operation is successful (the password has been changed), respond with the
appropriate case:

* If the request prefers `text/html`:

  * If `autoLogin` is `false`, redirect the user to
    `stormpath.web.changePassword.nextUri`

  * If `autoLogin` is `true`, log the user in (create the Oauth2 cookies) and
    redirect the user to `stormpath.web.login.nextUri`

* If the request prefers `application/json`:

  * If `autoLogin` is `false`, respond with `200 OK` and an empty body.

  * If `autoLogin` is `true`, log the user in (create the Oauth2 cookies) and
    respond with the authentication response:

    ```
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8

    {
      account: {
        // the account that was created, stripped of all non-root properties
      }
    }
    ```

**Error Responses**

If an error is encountered, respond with the appropriate case:

* If the request is `Accept: text/html`:

  * If the error is a password policy validation error, re-render the change
    password form and display the error to the user

  * If the error is an invalid or expired token error, redirect the user to
    `stormpath.web.changePassword.errorUri`

* If the request is `Accept: application/json`:

  * Respond with the JSON error from the API, according to the [Error Handling][]
    specification.

### Options

```yaml
stormpath:
  web:
    changePassword:
      enabled: null
      autoLogin: false
      uri: "/change"
      errorUri: "/forgot?status=invalid_sptoken"
      nextUri: "/login?status=reset"
      view: "change-password"

```



#### enabled

Default: `null`

Unless explicitly set to `false`, this endpoint must be automatically enabled if
the default account store for the defined Stormpath application has the password
reset workflow enabled.

<a href="#top">Back to Top</a>


#### autoLogin

Default: `false`

If `true`, then once a user has successfully reset their password, they will be
automatically logged in and redirected to `stormpath.web.login.nextUri`.


<a href="#top">Back to Top</a>


#### uri

Default: `/change`

The URI that we'll attach an interceptor to for requests, if `enabled` is
`true`.

<a href="#top">Back to Top</a>


#### <a name="change-error-uri"></a> errorUri

Default: `/forgot?status=invalid_sptoken`

This is the URI we'll redirect a user to if they have arrived with an invalid
`sptoken`.

<a href="#top">Back to Top</a>


#### nextUri

Default: `/login?status=reset`

This is the URI we'll redirect the user to after they have successfully reset
their account password.

<a href="#top">Back to Top</a>


#### view

A string key which identifies the view template that should be used.

<a href="#top">Back to Top</a>

[Error Handling]: error-handling.md
