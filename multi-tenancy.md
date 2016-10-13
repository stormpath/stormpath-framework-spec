# Multi Tenancy

This document describes how our framework integrations should support our multi-tenancy feature, with a sane set of defaults.  At the moment we are focusing on the subdomain-based used case.

## Subdomain-Based Multi-Tenancy With Organizations

The following design guidelines must be followed when adding multi-tenancy features to a framework integration:

* This is an opt-in configuration, the developer must use `stormpath.web.multiTenancy=true` to declare that they want to opt-in to our default organization-based multi-tenant solution.

* On login, registration, email verification, and password reset, we need to resolve which organization should be used for the operation (all of these REST API operations accept an account store as an optional parameter, where the account store is identified by `nameKey` or `href`).  

* The resolution should be achieved with an "Organization Resolver".  The developer should be able to provide their own resolver, but our framework should provide a default resolver.  

* The resolver type/interface should have the following characteristics:

    - It receives a HTTP request for inspection (some may also provide the response, if available).
    - It returns an Organization if one can be resolved.
    - It attaches the resolved Organization to the request, so that the developer can make use of this context.

* The default organization resolver should follow this procedure to determine the organization:

    - If `stormpath.web.multiTenancy.useSubDomain=true` and `stormpath.web.domainName` is defined, e.g. `example.com`, then a request with the `Host` header specified as `org-a.example.com` should return the `Organization` that has the `nameKey` of `org-a`.
    - If the request provides an access token (it is an authenticated request), and the token has an `org` claim, then this value indicates the organization that the user authenticated against, and this is can be uses as the organization context.  But if `stormpath.web.multiTenancy.useSubDomain=true`, we must also assert that the request is for a subdomain that matches the same organization as the token.  If this is not true the request should be rejected (401).
    - As a final fallback, if the request body has a field of `organizationNameKey=org-a`, then the `Organization` which has the `nameKey` of `org-a` should be returned.  This should only be done when user is posting a form field that our framework handles, and that form needs the organization context.
    - The subdomain value should take precedence over the form value if both are provided.    
    


* If `stormpath.web.multiTenancy=true`, and the organization cannot be resolved, AND the developer has disabled the `organizationNameKey` form fields, then the operation (login, etc) will be posted against the application per normal.


* When processing a email verification request:
    - If the key has expired or cannot be found, and an organization cannot be resolved, redirect the user to `<domainName>/verify`, and render a field that allows them to specify their organization name.
    - If the key is valid, the REST API response needs to include the organization name key on the response, so that we can redirect the user to `<orgNameKey>.<domainName><verifyEmail.nextUri|login.nextUri>` - *REQUIRES REST API CHANGE*

* When processing a password reset request:
    - If the key has expired or cannot be found, and an organization cannot be resolved, redirect the user to `<domainName>/forgot`, and render a field that allows them to specify their organization name.
    - If the key is valid, the REST API response needs to include the organization name key on the response, so that we can redirect the user to `<orgNameKey>.<domainName><changePassword.nextUri|login.nextUri>` - *REQUIRES REST API CHANGE*

* When the user submits an organization name key, and that organization does not exist, the error message to the end user should not leak information about organizations that do or do not exist.  In this situation the error message should read to the effect "Username or password is invalid, or Organization does not exist".

## Future Use Cases

Here are some notes we have collected about possible future use cases:

### Self-service Organization Creation

AKA the "Slack" model:

- Org Registration always occurs on naked domain
- Login & forgot password may happen through naked domain or subdomain
    + Naked domain requires user to specify account store name key in form
- Requires a forgot-tenant solution 
- How is the new organization name resolved?
    + End user defines the name during signup
    + Automatically derived from the domain of the email address
