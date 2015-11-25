<a name="#top">Back to Top</a>

# Account Login

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service login of user accounts.

If enabled by `stormpath.web.login.enabled`, our library MUST intercept incoming
requests for `stormpath.web.login.uri`.

GET requests must:

* Serve a default HTML page with a login form, if the request is type is
  `Accept: text/html` (read down for default form description).

* Serve a view model, which describes the login form, if the request is type is
  `Accept: application/json` (read down for view model).

POST requests must:

* Handle a POST request from the default HTML login form or from a JSON client.

## <a name="Options"></a> Options

The default options for this feature are:

```yaml
stormpath:
  web:
    login:
      enabled: true
      autoRedirect: true
      uri: "/login"
      nextUri: "/"
      view: "login"
```

#### <a name="enabled"></a> enabled

Default: `true`

If `true` this feature will be enabled and our library will intercept requests
at for `uri`.

If `false` this feature is disabled and the base framework will be responsible
for the `uri`, likely resulting in a 404 Not Found error.

**NOTE**: If this feature is enabled, and no Account Stores are mapped to this
Application -- then throw an error during framework initialization as there are
no possible ways for a user to authenticate.

#### <a name="uri"></a> uri

Default: `/login`

This URI that our integration should bind to for handling GET and POST requests
for the `uri`.

#### <a name="nextUri"></a> nextUri

Default: `/`

Where to send the user after successful login.  This value can be overridden on
a per-request basis, if the parameter `?next=uri` is provide on the POST.

#### <a name="autoRedirect"></a> autoRedirect

Default: `true`

If enabled, and a user already has a valid session (Oauth2 token cookies),
instead of re-rendering the login page we will redirect the user to the URL
specified by `nextUri`.

If disabled, we won't redirect the user anywhere and will simply re-render the
login page.  However -- in this case we will *also* destroy any existing OAuth2
token cookies.  This ensures that odd edge cases won't occur wherein a user is
viewing a login page but can see their account information in some place (like a
menu bar).

#### <a name="view"></a> view

Default: 'register'

A string key which identifies the view template that should be used.  The
default value may look different for your framework.  The point of this value
is to allow the developer to override our default view with their own.

<a href="#top">Back to Top</a>



## Default HTML Login Form

The form MUST:

* Require an email address / username AND password.

* Render social login buttons, if social provider account stores exist.

* Render context specific messages, depending on the status query parameter
  (see ["Status Messages"](#status-messages) section).



## Login View Model

The login view model should be returned to the client if the GET request is
`Accept: application/json`.  This is for front-end clients that need to
dynamically know how to render the login form.

The model should have:

* A list of fields, as defined by `stormpath.web.login.fields`, and ordered by
  `stormpath.web.login.fieldOrder`.  Fields should only be in the list if their
  `enabled` property is `true`.  As such the enabled property can be omitted
  from each list element.

* A list of providers, such as social providers, and the required information
  to render a UI component for that provider.  At the moment we only have social
  providers, which simply have a button, so all that is needed is the `clientId`
  from the given directory configuration.  This is needed by the front-end
  client to provide the popup-based login flows (see [social.md][]).

Example view model definition:

```javascript
{
  "fields": [
    {
      "name": "username",
      "placeholder": "Username or Email",
      "required": true,
      "type": "text"
    },
    {
      "name": "password",
      "placeholder": "Password",
      "required": true,
      "type": "password"
    }
  ],
  "providers": {
    "google": {
      "clientId": "xxxx"
    }
  }
}
```


## <a name="POST_Body_Format"></a> POST Body Format

This endpoint accepts password based login, and social login.  If the POST is
coming from a HTML based login form, the data will be submitted as
`application/x-www-form-urlencoded`.  Front-end applications will post the data
as `application/json`.

**Password-based login**

For example, a form-based post would look like this:

```x-www-form-urlencoded
username=robert@stormpath.com&
password=mypassword
```

* The `username` field can be either a username or email.

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

For both of these cases, if the error is from the Stormpath REST API, then send
the `userMessage` property of that error.

**For HTML responses:**

For any errors, the response should be a 200 OK and the form should be
re-rendered with a UX that indicates which field is in error and what can be
done to fix the problem.

**For JSON responses:**

Send a 400 JSON response where the body is of the format `{ error: 'user
friendly error message' }`.

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



## <a name="GET_Request_Handling"></a> GET Request Handling

This describes how we render the login page after receiving a GET request.

If the request type is HTML, we should render a login page with a form that
accepts either a username or email address and a password.  It should render
the login buttons for social providers, if configured (see [social.md][]).

If the request type is JSON, send 405.

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
