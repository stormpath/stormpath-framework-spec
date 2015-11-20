# Current User Context

This support is for front-end clients such as AngularJs.  This endpoint allows
the front-end application to fetch the account object of the currently
authenticated user.

We must provide this endpoint because, for security reasons, we store our
access tokens in `HttpOnly` cookies which cannot be read by the JavaScript
environment.

### Configuration Options

This route is enabled by default at the `/me` uri, but this endpoint can be
changed or disabled entirely with these options:

```yaml
web:
  me:
    enabled: true
    uri: "/me"
```

### Endpoint Response

This endpoint should always respond with `Content-Type: application/json`, and
the body should be the JSON representation of the currently authenticated user:

```json
{
  "href": "https://api.stormpath.com/v1/accounts/xxx",
  "username": "robert@stormpath.com",
  "email": "robert@stormpath.com",
  "givenName": "Robert",
  "middleName": null,
  "surname": "Damphousse",
  "fullName": "Robert Damphousse",
  "status": "ENABLED",
  "createdAt": "2014-04-07T16:38:44.000Z",
  "modifiedAt": "2014-08-28T22:35:10.000Z",
  "emailVerificationToken": null,
  "customData": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/customData"
  },
  "providerData": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/providerData"
  },
  "directory": {
    "href": "https://api.stormpath.com/v1/directories/xxx"
  },
  "tenant": {
    "href": "https://api.stormpath.com/v1/tenants/xxx"
  },
  "groups": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/groups"
  },
  "applications": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/applications"
  },
  "groupMemberships": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/groupMemberships"
  },
  "apiKeys": {
    "href": "https://api.stormpath.com/v1/accounts/xxx/apiKeys"
  }
}
```

### Account Expansion

If a request is made to `/me?expand=X`, the query strings should be passed to
our underlying SDK.  This allows the developer to expand the necessary resoures
on the account object
