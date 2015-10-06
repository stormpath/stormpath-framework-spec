# Social Provider Integrations

## Table of Contents

* [Options](#Options)
  * [`<providerId>`](#providerId)
  * [uri](#uri)

## Feature Description

The framework integration should provide convenience support for the Social
Login providers that the Stormpath API supports.  At the time of writing we
support:

* Facebook
* Github
* Google
* LinkedIn

For server-side rendered pages, we should implement the page-based redirect flow
with the provider.  For front-end integrations, use the pop-up window approach.

In both situations, the default login form should render the login buttons for
any social provider that is configured.  A good example of this is ID Site.

## <a name="Options"></a> Options

This object is built at runtime by parsing the account stores that are mapped to
the Stormpath application that is provied to the framework.  It is attached to
`config.web.socialProviders`. All callbacks URLs follow the form
`/callbacks/:providerId`

In the example below, there is only one account store that is a social account
store and it is a Google account store.

### <a name="providerId"></a> &lt;providerId&gt;

For each account store that is a social provider, it should have a key in
this object.  The key should list the callback URI and the client ID and client
secret.

```json
{
  "socialProviders": {
    "uri": "/oauth/providers",
    "google": {
      "callbackUri": "/callbacks/google",
      "clientId": "xxxx",
      "clientSecret": "xxxx",
      "enabled": "true"
    }
  }
}
```

### <a name="uri"></a> uri

This URI will be consumed by front-end clients, as they need to know which
providers are available for login and what the client ID is for the provider (
the client ID is used by the provider's front-end JavaScript framework).

```json
{
  "google": {
    "clientID": "xxxx"
  }
}
```

<a href="#top">Back to Top</a>