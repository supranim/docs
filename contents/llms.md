# Supranim Documentation

> Supranim is a full-stack web framework for the Nim programming language. It ships a high-performance HTTP and WebSocket server built on powpow (a 100% Nim event loop), a macro-based ORM (Ozark) with compile-time checked SQL for PostgreSQL, a template engine (Tim), sessions, event listeners, background tasks, and a CLI tool (Supra) for bootstrapping projects from starter kits.

Supranim is designed to be fast, modern and simple to use. Routes are defined in a `routes.nim` file and are automatically linked to controller procs by name (e.g. `get "/"` → `getHomepage`). Controllers, models, services and middleware are auto-discovered at compile time, so there is minimal boilerplate. The framework has no C build dependencies: the HTTP server (powpow) and the cryptography layer (nimcypher, a pure-Nim port of Monocypher) are both implemented in Nim. The PostgreSQL driver requires the libpq client library at runtime.

## Getting Started

- [Introduction](https://supranim.com/): Overview of Supranim and its key features.
- [Installation](https://supranim.com/install): Install Supranim and the Supra CLI with Nimble. Requires Nim 2.2.10+.
- [Starter Kits](https://supranim.com/starter-kits): Pre-built project templates with authentication, session management and a dashboard.
- [Supra CLI](https://supranim.com/supra): The official CLI for creating projects (`supra init`) and bundling static assets (`supra bundle.assets`).
- [Dependencies](https://supranim.com/dependencies): The Nim packages used by Supranim and the runtime libpq requirement.
- [Deployment](https://supranim.com/deployment): Reverse proxies, SSL/TLS and release builds.
- [License](https://supranim.com/license): LGPL-3.0-or-later.

## The Concept

- [Project Structure](https://supranim.com/concept/project-structure): The standard `src/` layout: config, controllers, models, services, templates and routes.
- [Configuration](https://supranim.com/concept/configuration): YAML configuration files, `.env.yml`, and accessing config via `App.config(...)`.
- [Service Providers](https://supranim.com/concept/providers): Registering services as Singleton, Channel, WebService or ThreadService.

## Basics

- [Routing](https://supranim.com/basics/routing): Define routes and route parameters (`id`, `slug`, `uuid`, `semver`, ...), multi-method routes, route groups, middleware and afterware.
- [Middleware](https://supranim.com/basics/middleware): `newMiddleware` / `newBaseMiddleware`, `next()` and `abort()`.
- [Controllers](https://supranim.com/basics/controller): `newController` / `ctrl`, responding with `respond` or `json`.
- [Templates](https://supranim.com/basics/templates): Rendering HTML views with the Tim template engine (layouts, partials, views).
- [Validation](https://supranim.com/basics/validation): Input validation with `pkg/bag`.
- [Backends](https://supranim.com/basics/backends): The powpow HTTP/WebSocket backend.
- [Other Setups](https://supranim.com/basics/other-setups): Low-level HTTP and WebSocket server APIs.

## Database (Ozark ORM)

- [Models](https://supranim.com/database/models): Define models with `newModel` and auto-loaded tables.
- [Query Builder](https://supranim.com/database/queries): Compile-time checked SQL: insert, select, where (with `whereLike`, `whereIn`, `orWhere`), `orderDescBy`, `limit`, update, delete and raw SQL.
- [Collection](https://supranim.com/database/collection): Work with query results (`isEmpty`, `len`, `get`, iteration).
- [Migrations](https://supranim.com/database/migrations): Current state and plan for a built-in migration system.

## Service Providers

- [Sessions](https://supranim.com/services/session): User sessions with the `supranim_session` package, `withSession`, auth middleware.
- [Tasks](https://supranim.com/services/tasks): Schedule and manage background tasks with `newTask`.
- [Events & Listeners](https://supranim.com/services/events): Register listeners in `src/service/event/listeners/`, emit events with `event().emit(...)`.
- [Http Limiter](https://supranim.com/services/limiter): Rate-limit HTTP requests.

For API references, see the generated documentation at https://supranim.github.io/ and the source repositories under https://github.com/supranim and https://github.com/openpeeps.
