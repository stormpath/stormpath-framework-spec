# Multi Tenancy

This document describes how our framework integrations should support our multi-tenancy feature, with a sane set of defaults.  At the moment we are focusing on the subdomain-based used case.

## Subdomain-Based Multi-Tenancy With Organizations

The following design guidelines must be followed when adding multi-tenancy features to a framework integration:

* This is an opt-in configuration, the developer must define these properties to declare that they want to opt-in to our default organization-based multi-tenant solution:
    -  `stormpath.web.multiTenancy=true` 
    -  `stormpath.web.domainName=my-company.com`

* On login, registration, email verification, and password reset, we need to resolve which organization should be used for the operation (all of these REST API operations accept an account store as an optional parameter, where the account store is identified by `nameKey` or `href`).  

* The resolution should be achieved with a "Organization Resolver".  The developer should be able to provide their own resolver, but our framework should provide a default resolver.  The point of this resolver is to provide our view controllers with the organization to be use with REST API requests.

* The default resolver should have the following characteristics:

    - It receives a HTTP request for inspection
    - It returns an Organization if one can be resolved from the request context.
    - It should only return an Organization that is mapped to the configured application.
    
* The default resolver should follow this procedure to determine the organization:

    - If the request `Host` header is `org-a.example.com` we return the `Organization` that has the `nameKey` of `org-a`.
    - If the request body has a field of `organizationNameKey=org-a`, then the `Organization` which has the `nameKey` of `org-a` should be returned.
    - The subdomain value should take precedence over the form value if both are provided.    

* If the user visits any view that requires organization context (e.g. login), and the subdomain does not resolve to a organization, we should redirect the user to `<stormpath.web.domainName>/<view>`, so that the user can provide their organization context.

* If the user visits the root domain to specify organization context, the form should only show the organization name key field.  The user must enter a valid organization name key.  An error is shown if the org is not valid.  If the org is valid we redirect them to the same view on the correct subdomain.

* When processing a email verification request:
    - If the key has expired or cannot be found, and an organization cannot be resolved, redirect the user to `<baseDomain>/verify`, and render a field that allows them to specify their organization name.
    - If the key is valid, the REST API response needs to include the organization name key on the response, so that we can redirect the user to `<orgNameKey>.<baseDomain><verifyEmail.nextUri|login.nextUri>` - *REQUIRES REST API CHANGE*

* When processing a password reset request:
    - If the key has expired or cannot be found, and an organization cannot be resolved, redirect the user to `<baseDomain>/forgot`, and render a field that allows them to specify their organization name.
    - If the key is valid, the REST API response needs to include the organization name key on the response, so that we can redirect the user to `<orgNameKey>.<baseDomain><changePassword.nextUri|login.nextUri>` - *REQUIRES REST API CHANGE*
    
* The framework should provide a convenience the allows the developer to know the authenticated organization of an authenticated request.  This should be done by looking at the `org` claim of the access token that was used to authenticate the request.

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