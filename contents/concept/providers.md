---
title: "Service Providers"
description: "Manage your application's services and dependencies with the Service Provider system."
tags: ["service providers", "providers", "services"]
---

Service Providers are a powerful feature in Supranim that allows you to manage your application's services in a clean and organized way. They are responsible for **registering services**, and **bootstrapping your application**.

## Types of Service Providers
Supranim provides multiple types of service providers. A service provider can be registered as a **Singleton**, **Channel**, **WebService** for standalone microservices or **ThreadService** for running thread-based REST API or WebSocket services.

### Singleton Service Provider
A Singleton Service Provider is a service that is instantiated only once and shared across the entire application. It is useful for services that maintain state or need to be accessed globally, such as a database connection or a cache.

### Channel Service Provider
A Channel Service is based on the `threading/channels` module and is designed for running background tasks or services that need to run concurrently with the main application. It allows you to create a channel that can be used to send messages between different parts of your application.

### WebService Provider
A WebService Provider is a standalone service that runs its own web server. It is useful for creating REST API microservices that need to run independently from the main application.