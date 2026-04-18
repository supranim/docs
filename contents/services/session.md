---
title: "Session Management"
description: "Supranim's Session Management for handling user sessions, including creation, management, and cleanup."
keywords: ["session management", "cookie", "authentication", "registration"]
---

## About
The Session Manager is responsible for handling user sessions, including creation, management, and cleanup. It provides a secure way to manage user authentication and maintain session state across requests.

## Features
- Create, validate and manage User Sessions
- Login, Register, Logout and Reset password control flows
- Tim Engine templates for rendering session-related views
- Cookie Handling: Manage session cookies for client-side storage of session tokens
- CSRF token generation and validation for enhanced security

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">
    If you're using any of the Supranim Starter Kits you can skip these steps as the Session Manager service is already installed and initialized in the starter kits.
  </div>
</div>


## Install this service
Install the Session Manager service using Nimble:
```
nimble install session
```

## Initialize Session Manager
To use this service in your Supranim application, you need to initialize it in your main application file:

```nim
App.services do:
  # other services...
  session.init()
```

## Session flows
The Session Manager provides a `withSession` template that you can use to wrap your logic that requires session handling. This template will automatically handle
session creation, validation, and cleanup for you.

Here, is an example of how to use the `withSession` template in a controller action:

```nim
import ../provider/session

ctrl getAccount:
  ## GET handler for rendering the account screen
  withSession do:
    echo userSession.getId()
```

### Authentication Middleware
The Session Manager service provides an authentication middleware that you can use to protect routes that require user authentication. You can apply this middleware to your routes as follows:

```nim
import ../provider/session

newMiddleware authenticate:
  ## Checks if the user is authenticated.
  ## Otherwise, it redirects to the login page
  withSession do:
    let userData = req.getClientData()
    if userSession.isAuthenticated():
      # continue to the next middleware
      next()
  
  # redirects to `GET /auth/login` page
  abort("/auth/login")
```

### API Reference
The API reference for this service: https://supranim.github.io/session