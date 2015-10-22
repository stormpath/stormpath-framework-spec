<a name="#top">Back to Top</a>

# Oauth2 Features

Stormpath provides two Oauth2 workflows, client credential and password.  In
this document we discuss how these should be exposed in the developer's web
application.



### OAuth2 Configuration Options

The OAuth2 endpoint can be configured, or disabled entirely:

```yaml
web:
  oauth2:
    enabled: true
    uri: "/oauth/token"
```

#### web.oauth2.enabled

By default we accept POSTS to this token uri and respond according to the
grant type.  If the developer sets this value to false we should not attach any
handler to this uri, allowing the framework to return it's standard 404
response.

If enabled and the grant type is not `client_credentials` or `password`, then
return `400 Unsupported Grant Type`.

## Client Credentials Grant Flow

The product guide for this feature can be found here:

https://docs.stormpath.com/guides/api-key-management/

In this workflow, an api key and secret is provisioned for a stormpath account.
These credentials can be exchanged for an access token by making a POST request
to `/oauth/token` on the web application.  The request must look like this:

````
POST /oauth/token
Authorization: Basic <base64UrlEncoded(apiKeyId:apiKeySecret)>

grant_type=client_credentials
````

The underlying Stormpath SDK is then used for generating the access token
and verifying that the API key & secret is valid, and that the account is not
disabled.  The response is an access token response, but this flow does not
return a refresh token (only an access token):

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "expires_in":3600,
  "token_type":"Bearer"
}
```

### Client Credentials Options

```yaml
web:
  oauth2:
    client_credentials:
      enabled: true
      accessToken:
        ttl: 3600
```

#### web.oauth2.client_credentials.enabled

If set to false, the endpoint should return `400 Unsupported Grant Type` if a
client requests this grant flow

#### web.oauth2.client_credentials.accessToken.ttl

Determines how long the issued token should be valid for, as seconds.  The SDK
should be accepting this option when it creates the access token for this flow.


## Password Grant Flow

The product guide for this feature can be found here:

http://docs.stormpath.com/guides/token-management/

In this workflow, an account can post their username and password to the
`/oauth/token` endpoint, with the following body data:

````
POST /oauth/token

grant_type=password
&username=<username>
&password=<password>
````

The framework should use the underlying SDK to exchange these credentials
for an access token response, using the `/oauth/token` endpoint of the
Stormpath Application that is specified in the configuration.

If the login is successful, we respond with an access token and refresh token.
The format of the response should be an OAuth2 access token response, which
looks like this:

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "expires_in":3600,
  "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
  "token_type":"Bearer"
}
```

## Password Grant Options

```yaml
web:
  oauth2:
    password:
      enabled: true
      validationStrategies: ['local','stormpath']
      accessToken:
        ttl: "PT1H"
      refreshToken:
        ttl: "P60D"

```


#### web.oauth2.password.enabled

If set to false, the endpoint should return `400 Unsupported Grant Type` if a
client requests this grant flow


#### web.oauth2.password.validationStrategies

Determines how we validate an access token.  There are two strategies:

* **local** - only verify the signature, expiration, and issuer

* **stormpath** - always validate against Stormpath, for additional checks
  such as Account and Application status.  This is the default, but the most
  consistent (will always be aware of Stormpath resource statuses).

It is the responsibility of the SDK to know how to implement these two
strategies.

#### web.oauth2.password.accessToken

The `ttl` value of this property is store in the OAuth Policy of the configured
Stormpath application.  The config parser should populate the value so that
it's available to the framework, but the developer cannot set this value through
static configuration.  It must be set through the REST API or by using the
Stormpath Admin Console.

#### web.oauth2.password.refreshToken

Same as the previous option