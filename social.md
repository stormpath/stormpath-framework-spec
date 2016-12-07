# Social Provider Integrations v2

## Feature Description

The framework integration should provide convenience support for the Social
Login providers that the Stormpath API supports.  At the time of writing we
support:

* Facebook
* GitHub
* Google
* LinkedIn

Coming Soon:

* Twitter
* Generic OAuth

## Background: Social Login Workflows

The framework integration interacts with Stormpath via the Stormpath Client API in a uniform way - regardless of the social provider.

The Stormpath Client API interacts with the Stormpath REST API which in turn, interacts directly with the social provider using its page-based redirect workflow.

All of these details are hidden from the framework integration. The framework integration can implement an ajax-based workflow or a page-based workflow as it sees fit.

There are two primary responsibilities of the framework integration:

1. Include a Client API `authorizeUri` link in its login model for each configured social provider
2. Validate and parse a Stormpath JWT assertion response

## Client API `authorizeUri` link

In order to interact with the Client API, the integration needs the `webConfig` from the backend `/application` endpoint:

`GET /applications/:applicationId?expand=webConfig`

This returns a response like:

```
{
    ...
    "webConfig": {
        ...
        "dnsLabel": "elegant-lynx",
        "domainName": "elegant-lynx.apps.dev.stormpath.io",
        ...
        "login": {
            "enabled": true
        },
        ...
        "status": "ENABLED",
        ...
    }
    ...
}
```

The conditions necessary to include the `authorizeUri` provider URL in the framework integration's `accountStores` login model are:

* A Social Provider Directory must be mapped to the Application
* webConfig.status = ENABLED
* webConfig.login.enabled = true

The `authorizeUri` has the following minimum format:

`https://<webConfig domain name>/authorize?response_type=stormpath&account_store_href=<url encoded social directory href>`

There are additional parameters that can be added to the `/authorize` URL as described [here](#optional-query-parameters).

How the integration includes the `authorizeUri` in its login model is an implementation detail left to the integration developers.

Two example approaches are presented below.

### Use the Client API login model

Once you have the `webConfig.domainName`, you can get the `/login` model from the Client API: `https://elegant-lynx.apps.dev.stormpath.io/login`.

The response from the Client API will look like this:

```
{
   "form":{
      ...
   },
   "accountStores":[
      {
         "authorizeUri":"https://<webConfig domain name>/authorize?response_type=stormpath_token&account_store_href=<url encoded linkedin dir href>",
         "name":"LinkedIn",
         "provider":{
            "href":"<linkedin dir href>/provider",
            "providerId":"linkedin",
            "clientId":"<client id>",
            "scope":"r_basicprofile r_emailaddress"
         },
         "href":"<linkedin dir href>"
      },
      {
         "authorizeUri":"https://<webConfig domain name>/authorize?response_type=stormpath_token&account_store_href=<url encoded facebook fir href>",
         "name":"Facebook Dir",
         "provider":{
            "href":"<facebook dir href>/provider",
            "providerId":"facebook",
            "clientId":"<client id>",
            "scope":"public_profile email"
         },
         "href":"<facebook dir href>"
      }
   ]
}
```

The available `/authorize` URL(s) can be used as-is to kick off the social login flow. There are additional parameters that can be added to the `/authorize` URL as described [here](#optional-query-parameters).

### Construct the `/authorize` URL

The integration has all the information is needs to build the `authorizeUri` for inclusion in its login model. This minimum required information is:

1. Client API domain name (obtained from the `webConfig` above)
2. Stormpath Social Provider Directory href

### Optional Query Parameters

Each of the query parameters below should be individually url encoded if included as part of the `/authorize` url.

#### `redirect_uri`

The `redirect_uri` query parameter indicates the "last leg" of the flow. It should be a fully qualified url and it *must* be in the list of `authorizedCallbackUris` from the `/applications/:appID` endpoint.

If this query parameter is not included on the `/authorize` URL, then the first element of the `authorizedCallbackUris` list associated with the Application will be used.

#### `scope`

A default `scope` for an individual social provider is automatically set on the backend. This can be overridden by providing the `scope` query parameter.

#### `state`

If the `state` parameter is included, it will be passed back in the final leg of the flow to your application via the `redirect_uri`.

This parameter can be used like a CSRF token. That is, during `/authorize` URL construction, you can save the `state` value and then compare it to the `state` value that's passed back in the final leg of the flow.

## `redirect_uri` response

The last leg of the social interaction is to send a `302` redirect to the browser. The `Location` will be the `redirect_uri` originally set in the `/authorize` URL at the beginning of the flow. It will include a `jwtResponse` query parameter as well as a `state` query parameter (if it was included at the beginning of the flow).

The JWT set as the `jwtResponse` has an `stt` header parameter of `assertion`.

### Verifying the `jwtResponse` JWT

The JWT in the `jwtResponse` is signed by the Client API. As such, the same signing key must be used to verify the JWT.

The `kid` header parameter identifies the key id used to sign the JWT. If it is *not* the same key id that the framework integration is using, then the signing key can be retrieved from the Client API.

The Client API signing key can be retrieved by using the Stormpath API endpoint specified in the `signingApiKey` of the Application's `webConfig`:

```
"webConfig": {
	...
    "signingApiKey": {
        "href": "https://<Stormpath API Host>/v1/apiKeys/<API Key ID>"
    },
    "status": "ENABLED",
	...
}
```

Retrieving this URL returns an api key response that includes the API Key Secret used to sign the `jwtResponse`:

```
{
    "account": {
        "href": "<account href>"
    },
    "description": "<description>",
    "href": "<apiKeys href>",
    "id": "<API Key ID>",
    "name": "<name>",
    "secret": "<API Key Secret>",
    "status": "ENABLED",
    "tenant": {
        "href": "<tenant href>"
    }
}
```

Once verified, the `jwtResponse` should be processed in the usual way by the framework integration for an AUTHENTICATED response as indicated [here](/login.md#-post-response-handling).

## Look and Feel

All of the integrations, regardless of language, should have a consistent look and feel for the rendered buttons for social providers.

<a href="#top">Back to Top</a>

[login]: login.md
[registration]: registration.md
