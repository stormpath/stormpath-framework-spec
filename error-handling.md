### Error Handling

Stormpath REST API error messages look like the following:

```
{
  "status": 400,
  "message": "Invalid username or password.",
  "code": 7100,
  "developerMessage": "Login attempt failed because the specified password is incorrect.",
  "moreInfo": "http://docs.stormpath.com/errors/7100"
}
```

The only properties that should ever be exposed to the end-user are the message
and status properties.  As such, this error would be sent to the end-user like
so:

```
{
  "status": 400,
  "message": "Invalid username or password."
}
```

### Error handling rules:

* When rendering errors to the end user via an HTML view, use the message
  property from the REST API (if available).

* When rendering errors as JSON:
  * Send the message and status properties ONLY.
  * Set the HTTP status of the response to the value of the API error response.
    If the API error does not have a status, then use 400.


* If the error is created by the framework integration (it is not from the REST
  API), then use status 400 and a human-friendly message string.

* If there are multiple errors for a given request, choose the most relevant one
  and respond with that single error.

* Unexpected errors, such as network communication errors with the Stormpath
  API, should be returned as a 500 Internal Server Error message.

* Error responses from the `/oauth/token` endpoint should pass ONLY the `error`
  and `message` properties.