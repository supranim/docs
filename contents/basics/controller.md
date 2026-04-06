---
title: "Controller"
description: "Create controllers to handle your application's routes and logic"
---

Controllers are the heart of your application, responsible for **handling incoming requests**, **processing data**, and **returning responses**. In this section, we will explore how to create and use controllers in your application.

## Create a Controller
You can create a new controller by using either `newController` macro, or the short-version `ctrl` template. Both serve the same purpose. Here is an example of how to create a controller using the `newController` macro:

```
newController getHomepage:
  respond("Welcome to the homepage!")
```

Basically, the `newController` macro will transform into the following proc
```
proc getHomepage(req: var Request, res: var Response) {.gcsafe.} =
  respond("Welcome to the homepage!")

# or, if you prefer the short version 👇

ctrl getHomepage:
  respond("Welcome to the homepage!")
```

There are multiple ways of sending a response from a controller. The `respond` template is a convenient way to send a text or HTML response. You can also use the `json` template to send a JSON response. For more information on how to pack a response, check the [Response section](/response) in the documentation.

Controllers are defined in the `src/controller` directory, and they are automatically loaded by the application.

The identifier name of the controller is linked to the route path. For example, a controller named `getHomepage` will be linked to the route path `get "/"`, and a controller named `getUserProfile` will be linked to the route path `get "/user/profile"`. For more information on how routes are linked to controllers, please refer to the [Routing](/routing) section.
