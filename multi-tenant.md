# Multi-Tenancy

We have three primary schemes for multi-tenancy:

* Single directory, multiple groups (group per tenant)
* Multiple directories (one directory per tenant)
* Organizations (one directory per organization)

What do we need to do in order to easily support these configurations in our
framework integrations?

1. The `/login` endpoint needs to accept an `accountStore` object, which can
have an `href` or `nameKey` attribute.  This will allow people to:
  * Target directories if they have decided to use the multiple directory approach
  * Target account stores by name key

  Note: if using the multiple directory approcah, it us up to the developer to
  decide how they want to resolve the account store href when the user is logging in


  When rendering the login form, we need to decide if we show a field for
   the account store name key.

    * If more than one org mapped to the application, show the field
      unless `stormpath.web.login.organizationNameKey.enabled` is `false`

    * If `stormpath.web.login.organizationNameKey.useSubDomain` is `true`,
      pull the name key from the submdomain in the URL and pre-fill the
      form field with this

   How the the ASNK can resolved?

      - subdomain (opt into this by default if more than one org exists for the application)

3. Registration should not be enabled until we have account creation policy

4. Same logic applies for the "forgot password" form as (3)

5. May need to specify the base URL of the application, `stormpath.web.baseDomain`