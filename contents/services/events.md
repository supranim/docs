---
title: "Event Emitter"
description: "Supranim's Event Emitter for handling custom events and listeners within the application."
keywords: ["event emitter", "custom events", "listeners", "pub/sub"]
---

## About the Event Emitter Service
The Event Emitter service in Supranim is designed to facilitate the handling of custom events and listeners within the application. It allows developers to define events, register listeners, and emit events in a structured manner, enabling a decoupled architecture and promoting better code organization.

All listeners are executed in the background, **allowing the application to continue parsing and processing other events without waiting for the completion of these tasks.**

Listeners can be registered for specific events, and when an event is emitted, all associated listeners are invoked. This service is particularly useful for implementing features such as **notifications**, **reset password requests**, **logging**, and other asynchronous operations.

## Getting Started
Install the [Event Emitter](https://github.com/supranim/emitter) service by importing it into your Supranim application via Nimble, then add it to your `.nimble` file:

```
requires "emitter >= 0.1.0"
```

Once installed, you can tell Supranim to initialize the autoloaded service in your application entry point, inside the `services` block:
```
App.services do:
  # Initialize the Event Emitter service
  events.init()
```
This will initialize a Singleton instance of the Event Emitter service, which can be accessed throughout your application to define listeners and emit events.

## Defining Events and Listeners
To define an event, you can create a new event type and register listeners for it. In Supranim, all listeners are automatically discovered at compile time, is enough to place a listener module inside the `src/service/listeners/` directory, for example:

```nim

listener "account.password.request":
  ## This listener handles password reset requests for user accounts.
  ## It listens for the "account.password.request" event and processes
  ## the request accordingly
  {.gcsafe.}:
    echo "Handle non-blocking tasks related to this event"
    sleep(3000) # pretend to do some work
    echo "Finished processing the event"
```

## Triggering Events
To trigger an event, you can use the `emit` function provided. For example, to emit the `account.password.request` event with some data:
```nim
event().emit("account.password.request", some(@["test@example.com"]))
```

## Standalone Usage
This service can also be used as a standalone Nimble package in any Nim project. It is **based on the Libevent library**, which provides a robust and efficient event handling mechanism. You can find the standalone package and its documentation on GitHub: https://github.com/supranim/emitter

### API Reference
The API reference for this service: https://supranim.github.io/emitter