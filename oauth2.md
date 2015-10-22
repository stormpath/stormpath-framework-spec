<a name="#top">Back to Top</a>

# Oauth2 Features

Stormpath provides two Oauth2 workflows:

### Client Credentials

In this workflow, an api key and secret is provisioned for an account.  These
credentials can be exchanged for an access token by making a POST request
to `/oauth/token`.  The request must contain the header
`Authorization: Basic <base64UrlEncoded(apiKeyId:apiKeySecret)` and the body
must contain `grant_type=client_credentials`

The underlyhing Stormpath SDK is responsible for generating the access token
and verifying that the API key & secret is valid, and that the account is not
disabled

The develoepr can specify these options for th client credentials workflow:

```yaml
web:
  oauth2:
    enabled: true
    uri: "/oauth/token"
    client_credentials:
      enabled: true
      accessToken:
        ttl: 3600
```

The `ttl` determines the expiration time of the token.  This is a value in seconds.

### Password

In this workflow, an account can post their username and password to the
`/oauth/token` endpoint, with the following body data:

```
grant_type=client_credentials
username=<username>
passwqord=<password>
```

If the login is successful, we respond with an access token and refresh token.

This feature should use the Application's `/oauth/token` endpoint, on the
Stormpath API, for creating tokens.  For more infomration please see:

http://docs.stormpath.com/guides/token-management/

The following options are available for this feature:

```yaml
web:
  oauth2:
    password:
      enabled: true
```


This allows the developer to turn off the password grant flow.  The application's
oauth policy does contain the ttl values for the access token and refresh
token, but they do not need to be duplicated here because the framework
integration does not need to deal with these values when creating tokens, it
simply invokes the REST API And gets back a token if the credentials are valid.