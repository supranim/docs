---
title: "Routing"
description: "Learn about routing in web development, including how to define routes, handle requests, and manage navigation in your web applications."
keywords: ["routing", "web development", "routes", "navigation", "request handling"]
---

Routing is a fundamental concept in web development that allows you to define how your web app responds to different URLs and HTTP methods. Supranim's routing system is designed to be flexible and easy to use

Routes are defined in the `routes.nim` file. This file is automatically included into your Supranim project at compile-time. The `routes.nim` file does not require any imports, and you can directly define your routes using the `routes` block.

## Route Definition
```
routes:
  get "/" 
```

## Auto-linked Routes
In Supranim, all routes are automatically linked to their coressponding handler functions based on their names. For example, a route defined as `get "/"` will automatically be linked to a handler function named `getHomepage`. This convention allows for a clean and organized way to manage your route handlers without needing to manually link them.

The auto-linking mechanism uses the **HTTP method** as a prefix and a camelCase version of the route path.

```
routes:
  get "/user/{name:slug}
    # This route autolinks to a handler named `getUserName`
    # where `name` is a dynamic route parameter of type `slug`
    # which is basically a regex pattern that matches URL-friendly
    # strings (e.g., "john-doe", "my-blog-post")
```

## Supported Route Parameters
Supranim supports dynamic route parameters, which can be defined using curly braces `{}` in the route path. You can specify the parameter type using a colon `:` followed by the type name. For example, `{name:slug}` defines a route parameter named `name` of type `slug`.
Supported parameter types include: 

`id` matches numeric strings.
```
[0-9]+
```

`slug` matches URL-friendly strings.
```
[0-9A-Za-z-_]+
```

`anySlug` matches any string, including `/` characters.
```
[0-9A-Za-z-_/]+
```

`alphaSlug` matches alphabetic strings.
```
[A-Za-z]+
```

`uuid` matches UUID strings.
```
[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}
```

`hex` matches hexadecimal strings.
```
[0-9a-fA-F]+
```

`word` matches one word characters.
```
\w+
```

`wordWithDots` matches word characters and dots.
```
[\w\.]+
```

`any` matches any string.
```
.+
```

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">
    Currently, Supranim does not support custom regex patterns for route parameters. However, we plan to add this feature in the future to allow for even more flexibility in defining route parameters.
  </div>
</div>

### Route multiple Methods
You can define a common route for multiple HTTP methods, for example when you want to have the same
route for both `GET` and `POST` requests. To do this, simply use a tuple of methods in the route definition, just like this

```
routes:
  (get, post) "/edit/{id:id}"
    # This route autolinks to two handlers: `getEditId` and `postEditId`
    # where `id` is a dynamic route parameter of type `id`
    # which matches numeric strings (e.g., "123", "456")
```

### Middleware & Afterware
In addition to defining routes, you can also use the `middleware` and `afterware` pragmas to specify functions that should be executed before or after the route handler, respectively. This allows you to easily manage tasks such as authentication, logging, or response modification in a clean and organized way.

```
routes:
  get "/dashboard" {.middleware: [authMiddleware].}
    # This route will execute the `authMiddleware` function before the `getDashboard` handler

  get "/profile" {.afterware: [logAfterware].}
    # This route will execute the `logAfterware` function after the `getProfile` handler
```

### Route Groups
Supranim also supports route groups, which allow you to group related routes together under a common path prefix. This is useful for organizing your routes and applying common middleware or afterware to a group of routes.

```
routes:
  group "/admin" {.middleware: [adminAuthMiddleware].}:
    get "/dashboard"
      # This route will be accessible at "/admin/dashboard" and will execute the `adminAuthMiddleware` before the `getAdminDashboard` handler

    get "/users"
      # This route will be accessible at "/admin/users" and will also execute the `adminAuthMiddleware` before the `getAdminUsers` handler
```