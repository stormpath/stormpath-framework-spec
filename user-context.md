# Current User Context

This endpoint supports rich browser clients such as AngularJs, and mobile
clients.  This endpoint allows the client application to fetch the account
object of the currently authenticated user.

We must provide this endpoint because, for security reasons, we don't allow the
client to store any information about the user.  It must be fetched from the
server at runtime, and the request must be authenticated.

### Configuration Options

This route is enabled by default at the `/me` uri, but this endpoint can be
changed or disabled entirely with these options:

```yaml
stormpath:
  web:
    me:
      enabled: true
      uri: "/me"
```

### Endpoint Response

This endpoint should always respond with `Content-Type: application/json`, and
the body should be the JSON representation of the currently authenticated user.

By default, all linked resources should be removed from the object.  However the
developer can opt-in to expansion through configuration (see next section). In
this situation the linked resource can be returned.

This endpoint must always send the following headers, so that the browser does
not cache this response:

```
Cache-Control: no-cache, no-store
Pragma: no-cache
```

Example default response body:

```json
{
  "account": {
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
    "emailVerificationToken": null
  }
}
```

### Expansion Options

The developer can opt-in to expanding the account's linked resources, for
example:

```yaml
stormpath:
  web:
    me:
      expand:
        groups: true
```
