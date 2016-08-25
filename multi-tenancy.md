## Multi Tenancy (Work In Progress)

Features that need to be implemented:

* On login, registration, and password reset, we should be able to dynamically post the login attempt to an application or organization.

* "AccountStoreResolver(httpRequest, httpResponse)" will be responsible for deriving the organization name key from the request, either from a subdomain, or from a form field.  The developer should be able to supply their on account store resolver.  Default resolver is org based, and assumes that you have defined a "base domain", and that a subdomain is the organization name key.  Other cases TBD.

* Developer needs to proivded with resolved accont store context

* Unresolved case: what if you're not using subdomains, and you need the end user to specify the organization that they are logging into?