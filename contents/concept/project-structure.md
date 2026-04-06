---
title: "Project Structure"
description: "Explore the organized project structure of Supranim, designed to help you navigate and understand the framework's components and architecture for efficient development."
tags: ["project structure", "architecture", "components"]
---

The project structure of Supranim is organized to provide a clear and intuitive layout for developers. It follows a modular design, allowing you to easily navigate through the different components and understand how they fit together to create a powerful web framework.

## The structure tree
Here's a simple representation of the Supranim project structure:

```
src/
├── config/
├── controller/
│   ├── pages.nim
│   └── errors.nim
├── model/
│   └── user.nim
├── service/
│   ├── database/
│   │   └── migrations/
│   │   │   └── 2024_01_01_000000_create_users_table.nim
│   ├── event/
│   ├── middleware/
│   │   └── auth.nim
│   └── provider/
├── storage/
├── views/
│   ├── layouts/
│   ├── partials/
│   └── pages/
```

### Configs
The `config` directory contains configuration files for the application, such as database settings, environment variables, and other application-specific configurations. All the configuration files are represented in **YAML** format for easy readability and management.

Go to the [Configs](https://supranim.com/docs/concept/configs) documentation for more details on how to define and manage your application configurations.

## Controller directory
The `controller` directory contains the controller files that handle the incoming HTTP requests and return responses. Depending on the size and complexity of your application, you can organize your controllers into subdirectories for better maintainability.

Check the [Controllers](https://supranim.com/docs/concept/controllers) documentation for more details on to define controllers and their actions.

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">Controller files are automatically loaded by Supranim, so you don't need to manually import them in your application.</div>
</div>

## Model directory
The `model` directory contains the model files that define the data structures and interactions with the database. Models are responsible for representing the data and providing methods to interact with it, such as querying, inserting, updating, and deleting records.

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">Model files are automatically loaded by the Ozark Database Manager service, and stored in a MacroCache table for compile-time access.</div>
</div>

## Services
The `service` directory contains the service files that provide additional functionality to the application. Services can include database services, event services, middleware, and providers. They are designed to be reusable components that can be easily integrated into your application.