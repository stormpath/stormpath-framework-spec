# Social Provider Integrations

## Feature Description

The framework integration should provide convenience support for the Social
Login providers that the Stormpath API supports.  At the time of writing we
support:

* Facebook
* GitHub
* Google
* LinkedIn

## Background: Social Login Workflows

Social providers typically implement two authentication workflows, which we will
describe in detail:

### Page-based Redirect Workflow

In this situation, the user leaves your login page by redirect and is taken to
the provider for authentication.  Once authentication is complete, the user is
redirected back to a "callback" URI on your website.  This callback will provide
an access token or access code as a query parameter.  Your server uses a
confidential keypair of the provider to validate the token/code, then retrieves
the account data of the authenticated user.

In order to support this workflow, the framework integration must:

* Parse the provider configuration of the specified Stormpath application (see
  next section).

* Provide callback handlers for this workflow.  See "Implementing Page-Based
  Workflows" in this document.

### AJAX-based Workflow

In this situation the user does not leave your login page.  Instead a popup
window is created, and the user authenticates with the provider in that window.
The popup window is created by a JavaScript API in the browser, and when the
user is done with the popup window (either by authenticating or rejecting the
prompt) the JavaScript API will call back to your JavaScript application.  If
the user has authenticated you will be provided with a token/code, which you
must send to your server and validate with your confidential provider
credentials.  At this point the workflow is identical to the page-base redirect
workflow.

In order to support this worklow, the framework integration must do the
following:

  * Parse the provider configuration of the specified Stormpath application (see
    next section).

  * Accept the access token or code via the login endpoint that this framework
    provides, see [login][] for implementation spec.

  * Render buttons for social login, on the registration and login views (or
    view model), see [login][] and [registration][].


## Parsing Provider Configuration

During framework bootstrap, the specified Stormpath application should be parsed
for it's account store mappings.  If any of these account stores are a social
provider, the following must be done:

* Render a login button on the login page (see [login][]).

* Create a callback handler for the provider's page-based redirect flow (see
  next section)


## Implementing Page-Based Workflows

The library must implement provider callbacks for all social provider
directories that are mapped to the specified Stormpath application.  The
callback URLs for each provider should take the form of:

> `/<stormpath.web.socialProviders.callbackRoot>/<providerId>`

Where `providerId` is the property that is found in the `provider` object of
the Stormpath directory, e.g. `google` or `facebook`.

When the callback URL is requested with a GET request, the user is being
redirected back to the server after authentication with the provider.  The
callback handler must complete the following tasks:

  * Parse the callback data from the provider to get the access token or code
  * Create or update the account in Stormpath, by making a call to the
    `application.getAccount()` method in the SDK of the framework language
  * Create the OAuth2 token cookies for the user
  * Redirect the user to `stormpath.web.login.nextUri`

If any of these tasks fail, an error should be immediately rendered to the user
and the next task should not be attempted.

The error should be displayed on the login form in the same way other login
errors are shown.

## Implementing Popup-Based Workflows

Single-page applications will need to query the server to determine which social
providers are configured, and which fields to render for the login or
registration views.  In either context, the front-end application should make
a GET request to the login or registration endpoint, to fetch the view model
as JSON.  The view model will contain the necessary information.  See [login][]
and [registration][] pages for the view model definitions.

<a href="#top">Back to Top</a>

[login]: login.md
[registration]: registration.md