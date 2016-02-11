## User-Agent Tracking

We need to collect usage metrics on our front-end integrations (Angular, React)
and our mobile integrations (iOS, Android).  As such, the following needs to be
implemented:

* In the front-end/mobile library, if a request to the framework is on the same
  domain (won't cause a cross-domain error), the library should add the
  following header:

  > X-Stormpath-Agent: stormpath-sdk-angularjs/0.9.0

* If the `X-Stormpath-Agent` header is seen on an incoming request, the
  framework integration should pass this value down to the SDK when making any
  REST API requests that are needed to fulfill the framework response.  The SDK
  should prepend this value to the `User-Agent` header that is sent to the
  Stormpath REST API.

Note: there will not be a 1-1 mapping from framework endpoint to REST API
endpoint. This is okay.  We aren't looking to get metrics on specific endpoint
requests.  Rather, we simply want to know if a tenant is using a given front-end
SDK or not.