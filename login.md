<a name="#top">Back to Top</a>

# Account Login

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service login of user accounts.

If enabled by `stormpath.web.login.enabled`, our library MUST intercept incoming
requests for `stormpath.web.login.uri`.

GET requests must:

* Serve a default HTML page with a login form, if the request type is
  `Accept: text/html` and `text/html` is defined in `stormpath.web.produces`.

* Serve the developer's Single Page Application, if `stormpath.web.spa.enabled`
  is `true` and `text/html` is defined in `stormpath.web.produces`.

* Serve the login view model if the request type is
  `Accept: application/json` and `application/json` is defined in
  `stormpath.web.produces`.



POST requests must:

* Handle a POST request from the default HTML login form or from a JSON client.

## <a name="Options"></a> Options

The default options for this feature are:

```yaml
stormpath:
  web:
    login:
      enabled: true
      uri: "/login"
      nextUri: "/"
      view: "login"
```

#### <a name="enabled"></a> enabled

Default: `true`

If `true`, this feature will be enabled and our library will intercept requests
for `uri`.

If `false`, this feature is disabled and the base framework will be responsible
for the `uri`, likely resulting in a 404 Not Found error.

**NOTE**: If this feature is enabled, and no Account Stores are mapped to this
Application -- then throw an error during framework initialization as there are
no possible ways for a user to authenticate.

#### <a name="uri"></a> uri

Default: `/login`

The URI that our integration should bind to for handling GET and POST requests
for the `uri`.

#### <a name="nextUri"></a> nextUri

Default: `/`

Where to send the user after successful login.  This value can be overridden on
a per-request basis, if the parameter `?next=uri` is provided on the POST.

#### <a name="view"></a> view

Default: `login`

A string key which identifies the view template that should be used.  The
default value may look different for your framework.  The point of this value
is to allow the developer to override our default view with their own.

<a href="#top">Back to Top</a>



## Default HTML Login Form

The form MUST:

* Require a login (email address or username) AND password.

* Render provider login buttons, if provider account stores are mapped to the
  specified Stormpath application.

* Render context specific messages, depending on the status query parameter
  (see ["Status Messages"](#status-messages) section).

* Sets [authenication cookies](https://github.com/stormpath/stormpath-framework-spec/blob/master/cookie-authentication.md) (OAuth 2.0 Access / Refresh Tokens) on successful authentication (username/password).


## Login View Model

The login view model should be returned to the client if the GET request
is `Accept: application/json` and `stormpath.web.produces` contains
`application/json`.  This is for front-end clients that need to dynamically know
how to render the login form.

The model should have:

* A list of fields, as defined by `stormpath.web.login.fields`, and ordered by
  `stormpath.web.login.fieldOrder`.  Fields should only be in the list if their
  `enabled` property is `true`.  As such the enabled property can be omitted
  from each list element.

* A list of providers, such as social providers or SAML providers.  Providers
  are found by looking at the account store mappings of the specified
  application.
  * Social providers will need to expose the `clientId`, as front-end
    applications will need this.
  * SAML providers need to provide the name of the directory, so that we know
    what text to use for the button, and the href, so that we can place this
    value into the SAML request (as a query parameter in the link that the
    button points to).
  * The ordering of this list should follow the ordering of the account store
    mappings.  **NOTE**: this may change in the future.


Example view model definition:

```javascript
{
  "form": {
    "fields": [
      {
        "label": "Username or Email",
        "name": "login",
        "placeholder": "Username or Email",
        "required": true,
        "type": "text"
      },
      {
        "label": "Password"
        "name": "password",
        "placeholder": "Password",
        "required": true,
        "type": "password"
      }
    ],
  },
  "accountStores": [
    {
      "href": "https://api.stormpath.com/v1/directories/whatev"
      "name": "Name of account store"
      "provider": {
        "href": "...",
        "providerId": "saml"
        //DO NOT EXPOSE ANY OTHER SAML DATA
      }
    },
    {
      "href": "https://api.stormpath.com/v1/directories/whatev2"
      "name": "Google"
      "provider": {
        "href": "href of the directory",
        "providerId": "google",
        "clientId": "WHATEVS",
        //DO NOT EXPOSE ANY SECRET HERE.  NO CLIENT SECRETS!
      }
    }
  ]
}
```


## <a name="POST_Body_Format"></a> POST Body Format

This endpoint accepts password based login, and social login.  If the POST is
coming from an HTML based login form, the data will be submitted as
`application/x-www-form-urlencoded`.  Front-end applications will post the data
as `application/json`.

**Password-based login**

For example, a form-based post would look like this:

```x-www-form-urlencoded
login=robert@stormpath.com&
password=mypassword
```

* The `login` field value can be either a username or email.

* If either field is omitted, an error will be raised and the page will be
re-rendered.

**Social login**

In this situation, the front-end client has obtained an access code or
authorization code from the provider.  The front-end client is now submitting
this information, along with our providerId for the provider.

```json
{
    "providerId": "google",
    "accessToken": "xxx",
    "code": "xxx"
}
```

* Only one of `accessToken` or `code` will be provided, both are listed
  for example purposes only.

* See [social.md][] for more information about social login.

<a href="#top">Back to Top</a>


##  <a name="POST_Error_Handling"></a> POST Error Handling

**For HTML responses:**

For any errors, the response should be a 200 OK and the form should be
re-rendered with a UX that indicates which field is in error and what can be
done to fix the problem.

**For JSON responses:**

* If the request is `Accept: application/json`, the response should be HTTP 400
and a JSON object of the format:

```javascript
{
  "errors": [
    {
      "message": "User-friendly error message"
    }
  ]
}
```

## <a name="POST_Response_Handling"></a> POST Response Handling

This describes how we handle the response, after an account has been
successfully authenticated.

**For HTML responses:**

Issue a 302 redirect to the `nextUri` and create a new user session.  If the
form post had a query parameter of `?next=url`, then redirect to that location
instead of the defined `nextUri`.

**For JSON responses:**

Send a 200 JSON body response, where the body contains the account object:

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  account: {
    // the account that was authenticated
  }
}
```

<a href="#top">Back to Top</a>



## Status Messages

The user may be redirected to this page from another workflow.  The redirect
will append a query parameter that tells you the context of the redirect.  The
query parameter name will be `status`.  For each status, the login page should
render the appropriate message above the login form:

* `?status=unverified` - the user has successfully registered, but their account
is unverified.  Message to show:

> Your account verification email has been sent!  Before you can log into your
  account, you need to activate your account by clicking the link we sent to
  your inbox.  Didn't get the email?
  `<a href="#{stormpath.web.verifyEmail.uri}">Click Here</a>`

* `?status=verified` - the user has successfully verified their account and can
  now login.  Message to show:

> Your Account Has Been Verified.  You may now login.

* `?status=created` - the user has successfully registered, and email
  verification is disabled so the user may login immediately.  Message to show:

> Your Account Has Been Created.  You may now login.

* `?status=forgot` - the user has submitted the forgot password form and we have
  sent them an email with a password reset link.  Message to show:

> Password Reset Requested.  If an account exists for the email provided, you
  will receive an email shortly.

* `?status=reset` - the user has finished the password reset workflow and has
  set a new password for their account.  They may now login with their new
  password.  Message to show:

> Password Reset Successfully.  You can now login with your new password.

<a href="#top">Back to Top</a>



[social.md]: social.md
