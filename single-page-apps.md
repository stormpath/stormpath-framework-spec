# Single Page Applications

Our integration needs to integrate well with Single Page Applications (SPAs),
such as AngularJS and React.  We've taken the approach that our frameworks
should not be involved with rendering these SPA assets, but they should be
configurable to support these primary developer stories:

* **I just want a JSON interface that my front-end library can use to integrate
  with Stormpath, I already have an asset pipeline for my SPA.**

* **I want the `/login` URL to serve my SPA, but when I post to it I want
  Stormpath to handle the login post.**

In order to achieve these stories, we need to give the developer a way to
"disable" the default HTML responses that we provide, e.g. to turn off our
default login and registration forms.  To achieve this we have the `produces`
option, which is a whitelist of content types that this library CAN produce,
and is ordered by preference.

As such, to achieve either of the stories above, the developer would remove
`text/html` from the default list, like this:


```yaml
stormpath:
  web:
    produces:
      - application/json
```

In this configuration, we do not serve our default HTML pages when we see
requests for URLs that we handle.  As such, it is expected that the developer
will be serving those assets themselves.

This relies on our [Content Negotiation Strategy][].


## View Models

The login and registration views are dynamic.  The registration form has
configurable fields, and both login and registration need to dynamically render
buttons for any providers (Google, Facebook, SAML) that are mapped to the
specified Stormpath application.

For that reason, we support JSON GET requests of the `/login` and `/register`
views.  The response is a view model that the client can use to render the
form appropriately.

Please see [login][] and [registration][] for specific information on each
model.


## Background: SPA "Routing"

Typical SPAs have an "entry", which is an HTML file (usually index.html) that
loads several JavaScript files and CSS files.  Once loaded, the application
will bootstrap.

"Routing" for SPAs refers to the strategy by which URLs map onto specific views
or states in the SPA.

For example, you may have URLs like this:

```
http://myapp.com
http://myapp.com/blog
http://myapp.com/blog/:postId
```

Or like this:

```
http://myapp.com
http://myapp.com/#blog
http://myapp.com/#blog/:postId
```

The latter is referred to as "hash bang" routing.  This is a simpler situation
and does not require coordination from the server.

The former is referred to as "HTML5" routing, as it leverages an HTML5 API in
the browser.  This API allows the user to navigate to absolute URLS, but the
browser will not make GET requests of the server for each URL.  The browser
merely updates the URL in the URL bar, and expects that the SPA will observe
this and render the appropriate view.  The SPA will likely make a JSON GET
request of the server, to load the data that is required to populate the view.

However, if the user hits the reload button (or visits the URL directly, from
another site) a GET request *will* be made of the server.  In this situation,
the server needs to render the entry for the SPA - so that the SPA can
bootstrap and then render the appropriate view.


[Content Negotiation Strategy]: requests.md#content-type-negotiation
[login]: login.md
[registration]: registration.md