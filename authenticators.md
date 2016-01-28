# Authenticators

The underlying SDKs provide Authenticators, which are used for authenticating
a user in various contexts.  A full list can be found here:

https://github.com/stormpath/stormpath-sdk-spec/blob/master/specifications/authenticators.md

Those authenticators have an interface which is *not* HTTP-specific, the
interfaces work with the necessary parameters for achieving authentication.

As such, our framework integrations will be responsible for taking an incoming
HTTP request, gathering the needed authentication parameters (from the body,
from HTTP headers, etc) and delegating to the necessary authenticator.

This work should be encapsulated into re-usable functions that can be exposed
in the public API of the framework.

The default set of functions that we want to provide are:

* BasicRequestAuthenticator
* OAuthBearerRequestAuthenticator
* OAuthClientCredentialsRequestAuthenticator
* OAuthPasswordRequestAuthenticator
* OAuthRefreshTokenRequestAuthenticator
* OAuthStormpathTokenRequestAuthenticator

We are evaluating the possibility of "smart" request authenticators, that can
take any request and delegate to the specific request authenticator.  For
example, a "OAuthRequestAuthenticator", which will delegate to the proper
OAuth request authenticator.