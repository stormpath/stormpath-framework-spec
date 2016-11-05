# Multi Tenancy

This document describes how our framework integrations should support our multi-tenancy feature, with a sane set of defaults.  At the moment we are focusing on the subdomain-based used case.

## Subdomain-Based Multi-Tenancy With Organizations

The following design guidelines must be followed when adding multi-tenancy features to a framework integration:

* This is an opt-in configuration, the developer must define these properties to declare that they want to opt-in to our default organization-based multi-tenant solution:

    ````yaml
    stormpath.web.multiTenancy.enabled: true
    stormpath.web.multiTenancy.strategy: "subodmain"
    stormpath.web.domainName: "my-company.com
    ````
If the first property is enabled, but the others are not present, this should be a configuration warning, and the feature should not be enabled.

* On login, registration, email verification, and password reset, we need to resolve which organization should be used for the operation (all of these REST API operations accept an account store as an optional parameter, where the account store is identified by `nameKey` or `href`).  

* The resolution should be achieved with a "Organization Resolver".  The developer should be able to provide their own resolver, but our framework should provide a default resolver.

### Default Organization Resolver

* The default resolver must have the following characteristics:

    - It receives a HTTP request for inspection
    - It returns an Organization if one can be resolved from the request context.
    - It only returns an Organization that is mapped to the configured application.
    - It attaches the resolved Organization to the request, so that the developer can make use of this context.
    - If the request `Host` header is `org-a.example.com` it returns the `Organization` that has the `nameKey` of `org-a`.
    - If the request body has a field of `organizationNameKey=org-a`, then the `Organization` which has the `nameKey` of `org-a` is returned.
    - The `Host` header value takes precedence over the form value if both are provided.    

* The default resolver may have the following characteristics:
    - It can be used as a standalone request filter/middleware component, so that the developer can make use of the resolution without opting in to the rest of the behavior that is described in this document.

### Changes to standard workflows

If the developer has opted in to the subdomain strategy, the framework should do the following:

* If the user requests a view that we control and the subdomain does not resolve to a organization, we should redirect the user to the parent domain, `<stormpath.web.domainName>/<view>`, so that the user can provide their organization context.

* If the user visits the root domain to specify organization context, the form should only show a field for specifying an organization name key.  The user must enter a valid organization name key.  An error is shown if the org is not valid.  If the org is valid we redirect them back to the same view on the correct subdomain.

### Email Veficiation & Password Reset

While it is possible to specify an account store when generating tokens for email verification and password reset, this context is not retained in the 
token/resource that is generated.  This means that we can't know which organization the user intends to use when they are arriving on the webapp with an sptoken.  We will likely fix this via the  REST API in the future, but in the meantime the following options are available to the developer:

* **Use a distinct directory per Organization**.  This will allow them to define a subdomain-specific Link Base URL in the email templates for the directory, so that the user will always land on the correct subdomain when they click the link in their email.

* **Use the parent domain as the Link Base Url**.  In this situation, the end user will always land on the parent domain with the `sptoken` in the URL.  The user must enter their organization name to get redirected to the correct subdomain, carrying the `sptoken` through, where they can complete the operation on the subdomain.

## Out of Scope

The following features are currently out of scope, but please use documentation to suggest implementation strategies, if possible:

* A "Forgot Organization" feature for the end-user.
* Self-service creation of new organizations.
