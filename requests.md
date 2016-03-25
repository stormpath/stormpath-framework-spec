## Request Behavior

This document describes how our framework integrations should treat incoming
requests and how it should respond, based on the nature of the request.

### Filter-Based approach

Our library should try to respond to the verbs and content types that it is
configured for, but "pass" on the request otherwise.  We take this approach so
that we don't interfere with the developers application.

### Disabled Routes

If the route is disabled in configuration, the request should be passed on to
the next component/filter of the underlying web framework handler, likely
resulting in a 404 error.

### Content-Type Negotiation

At every point where our library can render an HTML or JSON response, the
following decision tree must be followed:

The following logic applies to all incoming requests:

* If the request contains no Accept header, treat it as `Accept: */*`.

* If the request specifies `Accept: */*`, the first Content-Type in
  `stormpath:web:produces` will be returned.

* If the request specifies an Accept type that is in `stormpath:web:produces`,
  that Content-Type will be returned.

* If the request specifies an Accept type that is *not* in
  `stormpath:web:produces`, the integration should pass on the request

This can also be visualized as a flow diagram:


```
                   Is Accept Header
                   undefined or */* ?
                        |
                +-------+-------+
                |               |
               Yes              No
                |               |
           Serve first          |
           type in produces     |
           config               |
                                |
                          Is HTML preferred,
                          over JSON, in the
                          Accept Header?
                                |
                +---------------+---------------+
                |                               |
               Yes                              No
                |                               |
              Is HTML                         Is JSON in
            in produces?                     Accept Header?
                |                               |
        +-------+-------+               +-------+-------+
        |               |               |               |
       Yes              No             Yes              No
        |               |               |               |
   Serve the HTML       |             Is JSON           |
   view as defined      |           in produces?        |
   by <feature>.view    |               |               |
                        |       +-------+-------+       |
                        |       |               |       |
                        |      Yes              No      |
                        |       |               |       |
                        |   Serve JSON          |       |
                        |   view model          |       |
                        |                       |       |
                        +-----------------------+-------+
                        |
    Pass, developer needs to serve a SPA. Otherwise
    the framework will let this fall through to it's
    default 404 response.
```