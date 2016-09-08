## Multi Tenancy

The following design guidelines must be followed when adding multi-tenancy features to a framework integration:

* On login, registration, and password reset, we should be able to dynamically resolve which account store should be used for the operation (all of these REST API operations accept an account store as an optional parameter, where the account store is identified by `nameKey` or `href`).  

* The resolution should be achieved with an "Account Store Resolver", `AccountStoreResolver`, which must have a `getAccountStore` method with the signature of `accountStoreResolver.getAccountStore(httpRequest, httpResponse)` and must return an `AccountStore`, which must have either an `href` or `nameKey` property.

* The developer should be able to implement their own `AccountStoreResolver` and pass it to our library.
 
* The integration must provide a default resolver for organization-based multi-tenancy, named `DefaultOrganizationNameKeyResolver`, which is an implementation of `AccountStoreResolver`.  This resolver should have the following behavior:
    - If `stormpath.web.basePath` is defined, e.g. `example.com`, then a request with the `Host` header specified as `org-a.example.com` should return an `AccountStore` which has the `nameKey` of `org-a`.
    - If the request body has a field of `organizationNameKey=org-a`, then an `AccountStore` which has the `nameKey` of `org-a` should be returned.  This is used for cases where the end-user must manually specify the organization that they wish to login to, and the developer must also enable `stormpath.web.login.form.fields.organizationNameKey.enabled=true`.
    - If an `AccountStore` is resolved, it should be attached to the `httpRequest` so that the developer can use this context in their own code.