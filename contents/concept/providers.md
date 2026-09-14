---
title: "Service Providers"
description: "Manage your application's services and dependencies with the Service Provider system."
tags: ["service providers", "providers", "services"]
---

Service Providers are a powerful feature in Supranim that allows you to manage your application's services in a clean and organized way. They are responsible for **registering services**, and **bootstrapping your application**.

## Types of Service Providers
Supranim provides multiple types of service providers. A service provider can be registered as a **Singleton**, **Global**, **ThreadService**, **ThreadPoolService**, **WebService** or **UnixService** for standalone microservices and thread-based servers.

### Singleton Service Provider
A Singleton Service Provider is a service that is instantiated only once and shared across the entire application. It is useful for services that maintain state or need to be accessed globally, such as a database connection or a cache.

### Global Service Provider
A Global Service Provider is a namespace-style service without an instance type: only the `state`/`api` blocks are emitted, with no `get<Name>Instance` accessor. It is useful for grouping pure functions and constants under a service name.

### ThreadService Provider
A ThreadService runs user code in its own thread for as long as it wants and talks to the main application through the application-owned `Chan[ServiceMsg]`. It cannot be built standalone. (`ChannelService` is a deprecated alias of `ThreadService`.)

### ThreadPoolService Provider
A ThreadPoolService owns a `TaskManager` (see [Background Tasks & Queues](/services/tasks)): N anonymous workers run submitted jobs while callbacks fire serialized on the pool dispatch thread, and a private scheduler thread drives delayed, repeating and wall-clock tasks. Jobs are submitted via the `api` handles — there is no worker body and no `routes`, `ws` or `thread do:` block. It cannot be built standalone.

### WebService Provider
A WebService Provider is a standalone service that runs its own web server. It is useful for creating REST API microservices that need to run independently from the main application.

### UnixService Provider
A UnixService Provider is the same as a WebService Provider, but served over a Unix domain socket instead of TCP — ideal for fast local IPC. It supports `socketPath` and `socketMode` fields.