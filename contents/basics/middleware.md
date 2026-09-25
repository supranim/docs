---
title: "Middleware"
description: "Add middleware to your application to handle requests and responses."
---

Middleware is a powerful concept in web development that allows you to intercept and modify requests and responses in your application. In this section, we will explore how to use middleware in your application to enhance functionality and improve the user experience.

**Middleware files are located in the `src/service/middleware` directory, and they are automatically loaded by the application.**

## Create a Middleware
You can create a new middleware by using the `newMiddleware` macro. Inside the middleware use `next()` to continue to the next middleware or route handler, and `abort()` to stop the request and optionally redirect to another page.

Here is an example:
```nim
import supranim/middleware

newMiddleware authenticate:
  withSession do:
    let userData = req.getClientData()
    if userSession.isAuthenticated():
      next() # continue to the next middleware

  # if the auth fails will redirect to login page
  # using `abort` template (which also blocks code execution after call)
  abort("/auth/login")

  echo "Hello, Joy!" # so, this won't get printed!
```

### Attach Middleware to Routes
To attach a middleware to a route, you can use a pragma expression. For example to attach the `authenticate` middleware to `/account` route:
```nim
get "/account" {.middleware: [authenticate].}
  # GET route links to `getAccount` controller
  # and is protected by the `authenticate` middleware
```

This will ensure that the `authenticate` middleware runs before the route handler for `/account`, allowing you to perform authentication checks or other logic before processing the request. Sure, you can attach multiple middleware to a route by listing them in the `middleware` pragma:
```nim
get "/account" {.middleware: [authenticate, anotherMiddleware].}
```

## Base Middleware
The `baseMiddleware` is a special middleware that runs before all other middleware and route handlers. It is useful for tasks that need to be performed on every request, such as logging, setting headers, or handling CORS.

One example is to implement a base middleware that checks for a trailing slash in the URI and redirects to the correct URL if necessary:
```nim
import supranim/middleware

newBaseMiddleware uriChecker:
  ## Fix the trailing slash in the URI
  let path = req.getUriPath
  if path != "/" and path[^1] == '/':
    res.addHeader("Location", path[0..^2])
    req.resp(code = HttpCode(301), "", res.getHeaders())
```

In this example, the `uriChecker` middleware checks if the requested URI ends with a trailing slash (except for the root path "/"). If it does, it redirects the user to the same URI without the trailing slash using a 301 Moved Permanently response.

**Base middleware handlers are automatically** added to each request, so you don't need to manually include them in your routes. They will run **before any other middleware or route handlers**, ensuring that the necessary checks or modifications are applied to every request.

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">
    For performance reasons, avoid using Base Middleware for tasks that are not necessary on every request, as it will run for every incoming request to your application.
  </div>
</div>
