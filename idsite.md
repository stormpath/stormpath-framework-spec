# ID Site

ID Site is a hosted login & registration application that simplifies single-sign on (SSO) flows by providing a common place for redirects and cookie storage. If the developer has chosen to use ID Site, they likely have multiple Stormpath Applications that are redirecting the user to ID Site for authentication. When the user authenticates at ID Site, they are returned to the original application (service provider, AKA the SP) with Stormpath Assertion JWT. The application can verify this token and resolve the account that has authenticated.

If the developer needs to use ID Site, they will enable it with `stormpath.web.idSite.enabled`. `stormpath.web.callback.enabled` must also be true, otherwise the integration should throw an error. 

When ID Site is enabled, we should redirect the user to ID Site when the following URIs are accessed with a GET request:

* `stormpath.web.login.uri` - with a path of `stormpath.web.idSite.loginUri`
* `stormpath.web.logout.uri`
* `stormpath.web.register.uri` - with a path of `stormpath.web.idSite.registerUri`
* `stormpath.web.forgot.uri` - with a path of `stormpath.web.idSite.forgotUri`

In addition, ID Site needs `stormpath.web.callback.enabled` to be true, so we can pass back the Stormpath assertion JWT. 

# `/stormpathCallback`

A callback so Stormpath can pass information to the web application. This is currently being used for ID Site, but may be used in the future for SAML, Stormpath handled social login, webhooks, and other messages from Stormpath. 

## Configuration

    callback: 
      enabled: true
      uri: "/stormpathCallback"

## Request Types

* GET

## Content Type

* Any - always follows `text/html` logic.

## Parameters

`jwtResponse` - a [Stormpath JWT](http://docs.stormpath.com/guides/using-id-site/#handling-the-callback-to-your-application-from-id-site). This should be handled using the underlying SDK's Stormpath Token authenticator. 

## Response

The `jwtResponse` from ID Site can have three states, with the following response logic: 

**REGISTERED**

A new user was created. Follow the standard [post registration logic](registration.md#-post-response-handling) (like redirecting to the login page or autologin). 

**AUTHENTICATED**

A user logged in. Follow the standard [post login logic](login.md#-post-response-handling) (like setting cookies, and redirecting the user)

**LOGOUT**

A user logged out. Follow the standard [post logout logic](logout.md).

## Errors

If not a valid Stormpath JWT, render a `text/html` 401 Unauthorized` error. 