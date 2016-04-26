# Account Registration

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service registration of user accounts.

If enabled by `stormpath.web.register.enabled`, our library MUST inspect
incoming requests for `stormpath.web.register.uri`. and determine if a response
is needed, based on our [Content Negotiation Strategy][].

GET requests may:

* Serve a default HTML page with a dynamic registration form.

* Serve the dynamic registration JSON view model.

* Redirect the user to ID Site if `stormpath.web.idSite.enabled` is `true`. This
  should be done with the ID Site URL Builder in the SDK.

* Pass on the request.

POST requests may:

* Handle a POST request from the default HTML registration form or from a JSON
  client.

## Default Registration Form

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
      form:
        fields:
          givenName:
            enabled: true
            label: "First Name"
            placeholder: "First Name"
            required: true
            type: "text"
          middleName:
            enabled: false
            label: "Middle Name"
            placeholder: "Middle Name"
            required: true
            type: "text"
          surname:
            enabled: true
            label: "Last Name"
            placeholder: "Last Name"
            required: true
            type: "text"
          username:
            enabled: false
            label: "Username"
            placeholder: "Username"
            required: true
            type: "text"
          email:
            enabled: true
            label: "Email"
            placeholder: "Email"
            required: true
            type: "email"
          password:
            enabled: true
            label: "Password"
            placeholder: "Password"
            required: true
            type: "password"
          confirmPassword:
            enabled: false
            label: "Confirm Password"
            placeholder: "Confirm Password"
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

The URI that our integration should bind to for handling GET and POST requests
for the `uri`.


#### <a name="nextUri"></a> nextUri

Default: `/`

Where to send the user after successful registration, if [autoLogin](#autoLogin)
is `true`.


#### <a name="autoLogin"></a> autoLogin

If enabled, the user should be automatically logged in and redirected to the
`nextUri` after a successful registration.  This is only possible if the
default account store for the application has the email verification workflow
disabled.

**Not yet determined**: if auto login is not possible because email verification
is enabled, how should this error be surfaced?


#### <a name="fields"></a> form.fields

A map of field definitions.  The name of the property, e.g. `givenName`, should
be applied to the HTML input element as the `name` attribute.  Each field
definition must have the following properties:

* `enabled` - Determines if this field should be shown in the registration form.

* `label` - The value that is shown as a descriptive label for the field.

* `placeholder` - The placeholder attribute value for the HTML input element.

* `required` - Determines if this field should be required by the user.  This
  attribute should be applied to the input element, and validated when parsing
  a form post.

* `type` - the value to apply to the `type` attribute of HTML input element.

The intention of this configuration is to give the developer some simple control
over the basic things: labels, placeholders, and text input types.  Mileage will
vary with other HTML input types, as some of those types require additional
attributes that we do not define in our configuration at this time.

#### <a name="fieldOrder"></a> form.fieldOrder

A configurable array that allows the developer to change the order in which the
fields are rendered.


#### <a name="view"></a> view

Default: `register`

A string key which identifies the view template that should be used.  The
default value may look different for your framework.  The point of this value
is to allow the developer to override our default view with their own.

<a href="#top">Back to Top</a>



## Registration View Model

The registration view model should be returned when the request prefers
`application/json` and `stormpath.web.produces` contains `application/json`.
This is for front-end or mobile clients that need to dynamically know how to
render the registration form.

The model should have:

* A list of fields, as defined by `stormpath.web.register.form.fields`, and
  ordered by `stormpath.web.register.form.fieldOrder`.  Fields should only be in
  the list if their `enabled` property is `true`.  As such the enabled property
  can be omitted from each list element.

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
        "label": "First Name",
        "name": "givenName",
        "placeholder": "First Name",
        "required": true,
        "type": "text"
      },
      {
        "label": "Last Name",
        "name": "surname",
        "placeholder": "Last Name",
        "required": true,
        "type": "text"
      },
      {
        "label": "Email",
        "name": "email",
        "placeholder": "Email",
        "required": true,
        "type": "email"
      },
      {
        "label": "Password",
        "name": "password",
        "placeholder": "Password",
        "required": true,
        "type": "password"
      },
      {
        // ... other fields, as configured
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
        "scope": "email profile" // <-- from stormpath.web.social.google.scope
        //DO NOT EXPOSE ANY SECRET HERE.  NO CLIENT SECRETS!
      }
    }
  ]
}
```



## <a name="POST_Body_Format"></a> POST Body Format

The content type of the POST body may be `application/x-www-form-urlencoded` or
`application/json`, the framework integration should parse both.

This section uses JSON for example purposes.

### Required Fields

Every POST body MUST contain, at a minimum, these fields:

```javascript
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

### Custom Fields

The developer should be able to define their own fields in the
`stormpath.web.register.form.fields` block.  If the configured field has
`required: true`, the form parser should error if the user does not submit
the value.

Custom fields can be supplied on the root of the post body, or as child
properties of the custom data field.  Regardless of how they are supplied, the
library should apply these properties to the account's custom data object.

If the post contains a custom field that it NOT defined by the developer, the
library MUST reject the request with an error.  We do not allow arbitrary data
to be posted to an account's custom data object.

A post with custom fields may look like this:

```javascript
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

* If the request prefers `text/html`, the response should be 200 OK and the HTML
form should be re-rendered with a UX that indicates which field is in error and
what can be done to fix the problem.

* If the request prefers `application/json`, the response should be HTTP 400 and
a JSON object of the format:

  * Respond with the JSON error from the API, according to the [Error Handling][]
    specification.
  * If the error is not from the API (i.e. it's a form validation error) then
    create a custom error message, also following the [Error Handling][]
    specification.

## <a name="POST_Response_Handling"></a> POST Response Handling

This describes how we handle the response, if an account has been successfully
created.

* If the request prefers `application/json`, the response should be status 200
  with a JSON body, where the body contains the account object, but ONLY the
  root properties of the account should be presented.  All linked resources must
  be omitted, to prevent leakage of sensitive user data.  For example:

  ```
  HTTP/1.1 200 OK
  Content-Type: application/json; charset=utf-8

  {
    "account": {
      {
        "href": "https://api.stormpath.com/v1/accounts/xxx",
        "username": "foo",
        "modifiedAt": "2016-01-26T20:50:03.931Z",
        "status": "ENABLED",
        "createdAt": "2015-10-13T20:54:22.215Z",
        "email": "bar@stormpath.com",
        "middleName": null,
        "surname": "foo",
        "givenName": "bar"
        "fullName": "bar foo"
      }
    }
  }
  ```

* If the request prefers `text/html`, and...
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

[Content Negotiation Strategy]: requests.md#content-type-negotiation
[Error Handling]: error-handling.md
[login page status messages]: login.md#status-messages
[social]: social.md