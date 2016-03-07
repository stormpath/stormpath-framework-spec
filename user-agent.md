# User Agent

Each framework integration is responsible for generating a `User-Agent` string containing the library and runtime version. This string is passed down to the SDK layer to be sent to the Stormpath API.

## Format

The framework User-Agent string will follow this format:

```
[frontend agent token(s)?] [framework token] [web library token(s)]
```

A complete framework integration string will look like the following:

```
stormpath-sdk-angularjs/0.9.0 angular/1.4.7 express-stormpath/3.0.0 express/4.13.4
|--------- frontend agent tokens ---------| |-- framework token --| | web lib tkn |
```

The underlying SDK will [provide a mechanism](link*) for the framework integration to pass this string to the Client object.

## Token Groups

#### Frontend Agent Tokens *(if any)*

The integration will look for an `X-Stormpath-Agent from the frontend client, as described in [Agent Tracking](agent-tracking.md).

This group is optional. If not null or empty, the agent tokens should be **prepended** to the rest of the framework user agent string.

## Framework Token

The framework integration name and version separated by a slash '/'. For example, `express-stormpath/3.0.0`.

## Web Library Token

Any web libraries in use, and their versions. For example, `express/4.13.4`.
