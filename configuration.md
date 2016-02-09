# Configuration Options

## Overview

The framework integration has a set of configuration options that are specific
to web applications, this configuration extends the [base configuration for the
underlying SDK](https://github.com/stormpath/stormpath-sdk-spec/blob/master/specifications/config.md).

The entire framework configuration reference can be found in the
[web-config.yaml](web-config.yaml) file.

The configuration options inherently refer to high level features that the
framework integration should implement, you will find that each high level
feature has it's own markdown file in this repository.

## Application resolution

Our framework integrations will need to know which Stormpath Application should
be used (as all authentication features of Stormpath are tied to Applications).

The developer needs to specify the application, by name or href, using one of
these configuration properties:

```yaml
config:
  appliction:
    name: null
    href: null
```

If neither of these are specified, the integration should attempt to use the
tenant's only application (if the tenant only has one application).  **NOTE**:
all tenants have an application called "Stormpath", this application cannot be
modified and should be excluded when looking to see if the tenant only has one
application.

If the developer does not specify any application configuration, and more
applications exist beyond the default 'My Application', then an error should be
given to the developer - it should let them know that they need to define an
application.

The following exception messages should be used.

* Application could not be found when defined by HREF:

  > The provided application could not be found. The provided application
    href was: %href%

* Application could not be found when defined by name:

  > The provided application could not be found. The provided application
    name was: %name%

* Application was not defined, and could not be automatically resolved to the
  only application in the tenant:

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

* If `stormpath.web.register.autoLogin` is true, and email verification is
  enabled on the default account store, the following error should be given:

  > Invalid configuration: stormpath.web.register.autoLogin is true, but the
    default account store of the specified application has the email
    verification workflow enabled. Auto login is only possible if email
    verification is disabled. Please disable this workflow on this
    application's default account store.

* If `stormpath.web.register.enabled` is true, and there is no default account
  store mapped to the application:

  > No default account store is mapped to the specified application. A default
    account store is required for registration.
