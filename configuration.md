# Configuration Options

## Overview

The framework integration has a set of configuration options that are specific
to web applications, this configuration extends the [base configuration for the
underlying SDK](https://github.com/stormpath/stormpath-sdk-spec/blob/master/specifications/config.md).

The entire framework configuration reference can be found in the [web-config.yaml](web-config.yaml) file.

The configuration options inherently refer to high level features that the framework integration should implement, you will find
that each high level feature has it's own markdown file in this repository.

## Application resolution

Our framework intergations will need to know which Stormpath Application should be used (as all authentication features of Stormpath are tied to Applications).

The developer needs to specify the application, by name or href, using one of this configuration properties:

```yaml
config:
  appliction:
    name: null
    href: null
```

If neither of these are sepcified, the integration should attempt to use the tenant's only application (if the tenant only has one application).  **MOMTE**: all tenants have an application called "Stormpath", this application cannot be modified and shoud be excluded when looking to see if the tenant only has one application.

If the developer does not specify any application configuration, and more applications exist beyind the default 'My Application', then an error should be given to the developer - it should let them know that they need to define an explicit application.
