# User Agent

Each framework integration is responsible for generating a `User-Agent` string containing the library and runtime version, as well as optional client SDK versions. These strings are passed down to the SDK layer to be sent to the Stormpath API.

## Client SDK Tracking

We need to collect usage metrics on our front-end integrations (Angular, React)
and our mobile integrations (iOS, Android).  As such, the following needs to be
implemented:

* In the front-end/mobile library, if a request to the framework is on the same
  domain (won't cause a cross-domain error), the library should add the
  following header:

  > X-Stormpath-Agent: stormpath-sdk-angularjs/0.9.0 angular/1.4.7
  
  This value indicates the version of our library, and the version of the 
  framework that is being used.

* If the `X-Stormpath-Agent` header is seen on an incoming request, the
  framework integration should capture this value and include it in the string passed down to the SDK on any requests that are made to fulfill the framework response.

Note: there will not be a 1-1 mapping from framework endpoint to REST API
endpoint. This is okay.  We aren't looking to get metrics on specific endpoint
requests.  Rather, we simply want to know if a tenant is using a given front-end
SDK or not.

## Format

The framework `User-Agent` string will follow this format:

```
[client SDK token(s)?] [framework token] [web library token(s)]
```

A complete framework integration string will look like the following:

```
stormpath-sdk-angularjs/0.9.0 angular/1.4.7 express-stormpath/3.0.0 express/4.13.4
|----------- client SDK tokens -----------| |-- framework token --| | web lib tkn |
```

The underlying SDK will [provide a mechanism](https://github.com/stormpath/stormpath-sdk-spec/blob/master/specifications/user-agent.md) for the framework integration to pass this string to the Client object.

### Token Groups

#### Client SDK Tokens *(if any)*

The integration will look for an `X-Stormpath-Agent` from the frontend client, as described in Client SDK Tracking above.

This group is optional. If not null or empty, the agent tokens should be **prepended** to the rest of the framework user agent string.

#### Framework Token

The framework integration name and version separated by a slash '/'. For example, `express-stormpath/3.0.0`.

#### Web Library Token

Any web libraries in use, and their versions. For example, `express/4.13.4`.
