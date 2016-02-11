# Single Page Applications

Our integration needs to play nice with Single Page Applications (SPAs), such
as AngularJS and React.

Typical SPAs have an "entry", which is an HTML file (usually index.html) that
loads several JavaScript files and CSS files.  Once loaded, the application
will bootstrap.

## Background: Routing

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
another site) a GET request *will* be made of the server.  It is this situation
that becomes interesting.

In this situation, the server needs to render the entry for the SPA - so that
the SPA can bootstrap and then render the appropriate view.

This impacts our integration because we need to know how we should respond
when an `Accept: text/html` GET request comes in for `/login` and other URLs that
we bind to. Do we serve our default HTML pages, or do we serve the SPA entry?
Or do we ignore HTML entirely, and leave the SPA configuration up to the
developer?

## Configuration

We have the following default configuration options to support these various
routing and SPA use-cases:

```yaml
stormpath:
  web:
    produces:
      - text/html
      - application/json
    spa:
      enabled: false
      view: index
```

These can be combined to achieve the following use-cases:

**I don't want Stormpath to handle my SPA assets / I want just JSON**

In this situation, the developer likely wants "just the JSON api" to be
attached to their web application.  In this situation, the developer will
disable HTML responses of our library.  When this is true, we only respond to
requests where `Accept: application/json` is present.  The developer must use
this configuration:

```yaml
stormpath:
  web:
    produces:
      - application/json
```

**I want to use the default Stormpath routes with my SPA**

In this situation, the developer wants to use the default routes of `/login`,
etc, but they want their SPA to be served for these routes.

In this situation, they will need to enable the SPA option:

```yaml
stormpath:
  web:
    spa:
      enabled: true
      view: index  # or other value that indicates where the entry is
```

With this configuration, our integration should always render the SPA entry when
it wants to render an HTML response, which would otherwise be one of our
default HTML pages.

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

## React Flux

React Flux is a different approach which does not follow the separation of
concerns that we've been describing.  In this environment, the server is
rendering HTML responses which contain the initial view (already populated
with data) but also providing an API for getting more HTML views or JSON for
doing more view rendering client-side.

In this situation, the developer should disable HTML responses.  They won't
provide a SPA entry.  They are likely adding our module to an existing server,
purely for the JSON API that facilitates our user workflows.


[login]: login.md
[registration]: registration.md