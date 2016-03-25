## Request Behavior

This document describes how the framework should handle incoming requests at a high level.

### Disabled Routes

If the route is disabled in configuration, the request should be passed on to the next middleware component or underlying web framework handler (if applicable). Alternatively, the integration can respond with `404 Not Found`.

### Supported Verbs

If the route is enabled in configuration, but the request verb is not supported, the integration should respond with `405 Method Not Allowed`.

If the route is enabled in configuration, and the request verb is supported, the request should be handled normally as described by the appropriate section of the spec.

### `Accept` Header

The following logic applies to all incoming requests:

* If the request contains no Accept header, treat it as `Accept: */*`.

* If the request specifies `Accept: */*`, the first Content-Type in `stormpath:web:produces` will be returned.

* If the request specifies an Accept type that is in `stormpath:web:produces`, that Content-Type will be returned.

* If the request specifies an Accept type that is *not* in `stormpath:web:produces`, the integration should return `406 Not Acceptable`.
