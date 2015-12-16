<a name="#top">Back to Top</a>

# Password Reset

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service password reset of existing user accounts.

This feature has two endpoints:

  * The "forgot" endpoint, which is used for requesting a password reset email.

  * The "change" endpoint, which is used for setting a new password and is
    linked to from the password reset email.  This is where the user arrives
    with the `sptoken` that was sent in the email.


## Forgot Password Endpoint

This is the endpoint that a user will use if they want to request a password
reset email.

If the default account store of the stormapth application has the password reset
workflow enabled, and `stormpath.web.forgotPassword.enabled` is not set to
`false`, our library MUST intercept incoming requests at
`stormpath.web.forgotPassword.uri` and follow the request handling procedure
that is defined below.

### Request Handling

#### GET requests with `Accept: text/html`:

* If `stormpath.web.spa.enabled` is `false`, render a form which allows the
  user to request a password reset by entering their email address.  If the
  request contains the query `?status=invalid_sptoken`, a message should be
  shown above the form:

  > The password reset link you tried to use is no longer valid. Please request
    a new link from the form below.

* If `stormpath.web.spa.enabled` is `true`, return the SPA view.

#### POST Requests

This endpoint accepts a post from the password reset request form, and the only
field in this form is the `email` field.  The endpoint should accept the content
as `application/json` or `application/x-www-form-urlencoded`.

The format of the request is (JSON example):

```javascript
{
  "email": "foo@bar.com"
}
```

Regardless of whether or not the email address is associated with a user
account, we must do the following:

* If the request is `Accept: application/html`, redirect the user to
  `stormpath.web.forgot.nextUri`

* If the request is `Accept: application/json`, respond with `200 OK` and no
  body.

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


## Change Password Endpoint

This is the endpoint that a user will use if they have received a password
reset email and have clicked on the link in the email.  The link will point to
this endpoint, and contain the `sptoken` query parameter.

### Request Handling

#### GET requests with `Accept: text/html`:

* If there is a `?sptoken` query parameter in the URL:

  * Make an API request to Stormpath to validate the `sptoken`, but **do not**
    consume the token yet.

  * If the `sptoken` is invalid, redirect the user to
    `stormpath.web.changePassword.errorUri`.

  * If the `sptoken` is valid:

    * If `stormpath.web.spa.enabled` is `false`, render a page that contains a
     form which collects the user's new password (must include a password
     confirmation field).

    * If `stormpath.web.spa.enabled` is `true`, return the SPA view.

* If there isn't a `?sptoken` query parameter in the URL:

  * Redirect the user to `stormpath.web.forgotPassword.uri`

#### POST Handling

This endpoint accepts a post from the password change form.  The endpoint should
accept the request as `application/json` or `application/x-www-form-urlencoded`.

The format of the request is (JSON example):

```javascript
{
  "sptoken": "the sent token",
  "password": "new password"
}
```

The Stormpath API should be invoked to consume the token and reset the password.

**Success Responses**

If the operation is successful (the password has been changed), respond with the
appropriate case:

* If the request is `Accept: text/html`:

  * If `autoLogin` is `false`, redirect the user to
    `stormpath.web.changePassword.nextUri`

  * If `autoLogin` is `true`, log the user in (create the Oauth2 cookies) and
    redirect the user to `stormpath.web.login.nextUri`

* If the request is `Accept: application/json`:

  * If `autoLogin` is `false`, respond with `200 OK` and an empty body.

  * If `autoLogin` is `true`, log the user in (create the Oauth2 cookies) and
    respond with the authentication response:

    ```
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8

    {
      account: {
        // the account that was created
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

  * Respond with `400 Bad Request` and an error response:

    ```javascript
    {
      errors: [
        {
          message: 'user friendly error'
        }
      ]
    }
    ```

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