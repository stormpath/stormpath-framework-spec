# Handlers

The framework integration should support the following handlers in a way that
makes sense for the framework.  These handlers are convenience handlers that
allow developer to work with the current account or authentication context,
without having to re-implement our default HTTP response behaviors.

All of these handlers should give the developer the following control:

* Access to the HTTP request and HTTP response data
* Work with the account object of the context
* Allow the default HTTP response cycle to continue (default behavior)
* Manually end the HTTP response

The default HTTP response behavior for login and registration is defined in
[login.md](login.md) [registration.md](registration.md).

### Post-Registration Handler

This handler should be invoked immediately after an account is created via the
account registration process (i.e. the end user has just registered for a new
account).

**Common use cases**:

* Populate custom data on the user object.
* Synchronize an external database with Stormpath.

### Pre-Registration Handler

This handler should be invoked immediately before an account will be created.
The developer can modify the data that will be used to create the account.

**Common use cases**:

* Assert that the registration is permitted by inspecting the email address.
* Target an account store for the account creation attempt.
* Custom redirect based on business logic

### Post-Login Handler

This handler should be invoked immediately after an account has been
authenticated by a login form submission or OAuth2 authentication flow.

**Common use cases**:

* Login auditing
* Custom redirect based on business logic

### Pre-Login Handler

This handler should be invoked immediately before an account will be
authenticated. The developer can modify the data that will be used to
authenticate the account.


**Common use cases**:

* Determine which account store should be used for the login attempt