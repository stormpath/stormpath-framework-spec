# Configuration Options

## Overview

The framework integration has a set of configuration options that are specific
to web applications, this configuration extends the [base configuration loaded by the
underlying SDK](https://github.com/stormpath/stormpath-sdk-spec/blob/master/specifications/config.md).

The entire framework configuration reference can be found in the
[web-config.yaml](web-config.yaml) file.

The configuration options inherently refer to high level features that the
framework integration should implement, you will find that each high level
feature has it's own markdown file in this repository.

## Application validation

Our framework integrations will need to know which Stormpath Application should
be used (as all authentication features of Stormpath are tied to Applications).

The developer needs to specify the application, by name or href, using one of
these configuration properties:

```yaml
stormpath:
  application:
    name: null
    href: null
```

At startup, the integration should use these validation rules:

* If the key `application.href` exists, but does not contain the substring `/applications/`, throw an error. This is a simple sanity check to see if the href provided is a valid Stormpath application href. Use this exception message:

   > `'(the invalid href)' is not a valid Stormpath Application href.`
   
* If the key `application.href` exists, try to look up the Stormpath Application with that href. If the application can't be found, use this error message:

  > The provided application could not be found. The provided application
    href was: %href%

* If the key `application.name` exists, try to look up the Stormpath Application with that name. If the application can't be found, use this error message:

  > The provided application could not be found. The provided application
    name was: %name%

* If neither `application.href` or `application.name` are specified, attempt to resolve the application using the rules below.

## Application resolution

If an application is not specified in configuration, the integration should attempt to use the
tenant's only application (if the tenant only has one application).

**NOTE**: all tenants have an application called "Stormpath". This application cannot be
modified and should be **excluded** when looking to see if the tenant only has one
application.

* If a single application (besides "Stormpath") exists in the tenant, use that application.

* If zero or more than one applications (besides "Stormpath") exist in the tenant, throw this error:

  > Could not automatically resolve a Stormpath Application. Please specify
    your Stormpath Application in your configuration.

## Account Store Resolution

Account stores are required for login and registration.  As such, we need to
look at the account stores that are mapped to the application that is specified
by the developer.  The following cases need to be validated.

* If the specified application does not have any account stores mapped to it,
this exception should be thrown:

  > No account stores are mapped to the specified application.
    Account stores are required for login and registration.

* If `stormpath.web.register.enabled` is true, and there is no default account
  store mapped to the application:

  > No default account store is mapped to the specified application. A default
    account store is required for registration.

* If external provider directories are mapped to the application, but `stormpath.web.callback.enabled` has been set to `false`, the end user cannot complete the SAML workflow and this warning should be logged:

  > stormpath.web.callback.enabled is set to false, but SAML directories are mapped to this application.  Provider login workflows cannot be completed if this callback endpoint is disabled.
