
<a name="#top">Back to Top</a>

# Account Registration

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service registration of user accounts.

If enabled by `stormpath.web.register.enabled`, our library MUST intercept
incoming requests for `stormpath.web.register.uri`.

GET requests must:

* Serve a default HTML page with a registration form, if the request is type is
  `Accept: text/html` (read down for default form description).

POST requests must:

* Handle a POST request from the default HTML registration form or from a JSON
  client.

### Default Registration Form Description

The default registration form MUST:

* Require a first name (givenName).
* Require a last name (surname).
* Require an email address.
* Require a password.

The form MAY be configured by the developer, allowing them to:

* Make the `givenName` and `surname` fields optional (`required: false`).
* Remove the `givenName` and `surname` fields (`enabled: false`).
* Enable other default Stormpath Account fields (`username`, `middleName`).
* Enable the a password confirmation field.
* Define custom fields.

## <a name="Options"></a> Options

This is the set of default configuration for the registration feature, it
defines the default fields we support and which ones we want to show by
default.  If a field is `enabled`, it should be shown on the registration form.
If it's `required` then an error should be returned if the value is not
supplied by the user.

The developer can define custom fields by creating new field definitions in the
`fields` map.  New fields MUST have all the properties that you see on our
default fields.

```yaml
stormpath:
  web:
    register:
      enabled: true
      uri: "/register"
      nextUri: "/"
      autoLogin: false
      # autoLogin is possible only if the email verification feature is disabled
      # on the default default account store of the defined Stormpath
      # application.
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


#### <a name="enabled"></a> enabled

Default: `true`

This determines if the integration will handle the registration route, defined
by `stormpath.web.registration.uri`


#### <a name="uri"></a> uri

Default: `/register`

This URI that our integration should bind to for handling GET and POST requests
for the `uri`.


#### <a name="nextUri"></a> nextUri

Default: `/`

Where to send the user after successful registration, if [autoLogin](#autoLogin)
is `True`.


#### <a name="autoLogin"></a> autoLogin

If enabled, the user should be automatically logged in and redirected to the
`nextUri` after a successful registration.  This is only possible if the
default account store for the application has the email verification workflow
disabled.

**Not yet determined**: if auto login is not possible because email verification
is enabled, how should this error be surfaced?


#### <a name="fields"></a> fields

A map of field definitions.  Each field definition has the following properties:

* `enabled` - Determines if this field should be shown in the registration form.

* `required` - Determines if this field should be required by the user.

* `name` - The name to apply to the HTML input element.

* `placeholder` - The placeholder value for the HTML input element.

* `type` - the type of HTML input element (e.g. text, password).

If the developer provides any properties that are not defined in the above list,
the properties should be applied as HTML attributes to the input element.


#### <a name="fieldOrder"></a> fieldOrder

A configurable array that allows the developer to change the order in which the
fields are rendered.


#### <a name="view"></a> view

Default: 'register'

A string key which identifies the view template that should be used.  The
default value may look different for your framework.  The point of this value
is to allow the developer to override our default view with their own.

<a href="#top">Back to Top</a>



### Registration View Model

The registration view model should be returned to the client if the GET request
is `Accept: application/json`.  This is for front-end clients that need to
dynamically know how to render the registration form.

The model should have:

* A list of fields, as defined by `stormpath.web.register.fields`, and ordered
  by `stormpath.web.register.fieldOrder`.  Fields should only be in the list if
  their `enabled` property is `true`.  As such the enabled property can be
  omitted from each list element.

* A list of providers, such as social providers, and the required information
  to render a UI component for that provider.  At the moment we only have social
  providers, which simply have a button, so all that is needed is the `clientId`
  from the given directory configuration.  This is needed by the front-end
  client to provide the popup-based authentication flows (see [social][]).

Example view model definition:

```javascript
{
  "fields": [
    {
      "name": "givenName",
      "placeholder": "First Name",
      "required": true,
      "type": "text"
    },
    {
      "name": "surname",
      "placeholder": "Last Name",
      "required": true,
      "type": "text"
    },
    {
      "name": "email",
      "placeholder": "Email",
      "required": true,
      "type": "email"
    },
    {
      "name": "password",
      "placeholder": "Password",
      "required": true,
      "type": "password"
    },
    {
      // .. other fields, as configured
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

The content type of the POST may be `application/x-www-form-urlencoded` or
`application/json`, the framework should be configured to accept both.

This section uses JSON for example purposes.

### Required Fields

Every POST body MUST contain, at a minimum, these fields:

```json
{
    "email": "robert@stormpath.com",
    "password": "changeme"
}
```

If those fields are omitted, it is an error.  If any of the required fields are
omitted, that is also an error.

### givenName and surname fields

If the `givenName` and `surname` account properties are not specified by the
user in the POST, and the developer has configured these fields to be optional
(`required: false`), or has disabled them entirely (`enabled: false`), our
library MUST set the value to 'UNKNOWN' when we create the account via the
Stormpath REST API (as the API requires these fields to be populated).

### Configurable Fields

The developer should be able to define their own fields in the
`stormpath.web.register` configuration block.  If the configured field has
`required: true`, the form parser should error if the user does not submit
the value.

Custom fields can be supplied on the root of the post body, or as child
properties of the custom data field.  Regardless of how they are supplied, the
library should apply these properties to the account's custom data object.

If the post contains a custom field that it NOT defined by the developer, the
library MUST reject the request with an error.  We do not allow arbitrary data
to be posted to an account's custom data object.

A post with custom fields may look like this:

```json
{
    "email": "robert@stormpath.com",
    "password": "d",
    "customValue": "custom value can be on root object or in customData object",
    "customData": {
      "hello": "world"
    }
}
```

<a href="#top">Back to Top</a>

##  <a name="POST_Error_Handling"></a> POST Error Handling

For any errors indicated in this document, the following should happen:

* If the request is `Accept: text/html`, the response should be 200 OK and the
HTML form should be re-rendered with a UX that indicates which field is in error
and what can be done to fix the problem.

* If the request is `Accept: application/json`, the response should be a JSON
object of the format `{ error: String }`, where String is a user-friendly message
and the status of the response should be 400.  If the error is from the
Stormpath REST API, then send the `userMessage` property of that error.

## <a name="POST_Response_Handling"></a> POST Response Handling

This describes how we handle the response, if an account has been successfully
created.

* If the request is `Accept: application/json`, the response should be status
  200 with a JSON body, where the body contains the account object:

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  account: {
    // the account that was created
  }
}
```

* If the request is `Accept: text/html`, and..
  * The newly created account's status is ENABLED and [autoLogin](#autoLogin)
    is:
    * `True`: issue a 302 Redirect to the `nextUri` and log the user in (create
      the OAuth2 token cookies).
    * `False`: 302 redirect the user to `stormpath.web.login.uri`, and append
      the query parameter `?status=created` (see [login page status messages][]).

  * The newly created account status is UNVERIFIED, then 302 redirect the user
    to `stormpath.web.login.uri` and append the query parameter
    `?status=unverified` (see [login page status messages][]).

<a href="#top">Back to Top</a>

[login page status messages]: login.md#status-messages
[social]: social.md