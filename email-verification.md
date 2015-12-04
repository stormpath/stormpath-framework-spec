<a name="#top">Back to Top</a>

# Email Verification

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service verification of newly registered user accounts.

If the application's default account store has the email verification workflow
enabled, and `stormpath.web.verify.enabled` is not set to `false`, our library
MUST intercept incoming requests at `stormpath.web.verify.uri` and follow the
request handling procedure that is defined below.

## Request Handling

#### GET requests with `Accept: text/html`:

* If there is a `?sptoken` query parameter in the URL:

 * Attempt to verify the `sptoken`:

   * If the token is valid, redirect to `stormpath.web.verify.nextUri` and
     append `?status=verified`

   * If the token is invalid:

    * If `stormpath.web.spa.enabled` is `false`, render a form that allows them
      to request a new link by submitting their email address.  The form should
      show the error:

     > This verification link is no longer valid. Please request a new link from
       the form below.

    * If `stormpath.web.spa.enabled` is `true`, return the SPA view.

* If there isn't a `?sptoken` query parameter in the URL:

  * If `stormpath.web.spa.enabled` is `false`, render a form that allows them
    to request a new link by submitting their email address.

  * If `stormpath.web.spa.enabled` is `true`, return the SPA view.

#### GET requests with `Accept: application/json`:

* If there is a `?sptoken` query parameter in the URL:

 * Attempt to verify the `sptoken`:

  * If the token is valid, respond with `200 OK` and an empty body

  * If the token is invalid, respond with `400 Bad Request` and an error
    response:

    ```javascript
    {
      errors: [
        {
          message: 'user friendly error'
        }
      ]
    }
    ```

* If there is't an `?sptoken` query parameter in the URL, respond with this
  error:

  ```javascript
  {
    errors: [
      {
        message: 'sptoken not provided'
      }
    ]
  }
  ```

#### POST Requests

This endpoint accepts post requests from the form which allows you to request
a new link by entering your email address.  The endpoint should accept the
request as `application/json` and `application/x-www-form-urlencoded`.

The format of the request is (JSON example):

```javascript
{
  "email": "foo@bar.com"
}
```

Regardless of whether or not the email address is associated with a user
account, we should render a success message that says:

> If the email address you entered was associated with an account, you will
  receive an email from us shortly.

If the request is `Accept: application/json`, the status of the response should
be `200 OK` with no body.

## <a name="Options"></a> Options

```yaml
stormpath:
  web:
    # Unless verifyEmail.enabled is specifically set to false, the email
    # verification feature must be automatically enabled if the default account
    # store for the defined Stormpath application has the email verification
    # workflow enabled.
    verifyEmail:
      enabled: null
      uri: "/verify"
      nextUri: "/login"
      view: "verify"
```


#### enabled

Default: `null`

Unless explicitly set to false, the email verification feature must be
automatically enabled if the default account store for the defined Stormpath
application has the email verification workflow enabled.

<a href="#top">Back to Top</a>


#### uri

Default: `/verify`

The URI that we'll attach an interceptor to for GET and POST requests, if
`enabled` is `true`.

<a href="#top">Back to Top</a>


#### nextUri

Default: `/login`

Where to send the user after successful verification.

<a href="#top">Back to Top</a>

#### view

Default: `verify`

A string key which identifies the view template that should be used.  The
default value may look different for your framework.  The point of this value
is to allow the developer to override our default view with their own.

