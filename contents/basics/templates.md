---
title: "Templates"
description: "Powerful templating system for rendering HTML views with dynamic data",
tags: ["templates", "views", "rendering", "tim", "template engine"]
---

## Introduction to Templates
Templates are a crucial part of any web application, allowing you to render dynamic HTML views based on your application's data. In Supranim, we use the [Tim template engine](https://github.com/openpeeps/tim) as a Supranim service to handle all template rendering.

## About Tim Engine
Tim is a simple and efficient DSL (Domain Specific Language) for creating templates. It provides a clean syntax for embedding dynamic data into your HTML views, making it easy to create reusable and maintainable templates. **There is a Booyaka documentation site dedicated to Tim**, which you can find it here https://tim.openpeeps.dev

## Template Files
By default, Tim Engine looks for template files in the `src/templates` directory of your project. Inside this directory you can organize your templates in in `layouts`, `partials` and `views` subdirectories:

- `layouts`: This directory contains layout templates that define the overall structure of your pages. Layouts typically include common elements like headers, footers, and navigation bars.
- `partials`: This directory contains partial templates that can be included in other templates. Useful for reusable components (forms, buttons), or any other HTML snippets that you want to use across multiple pages.
- `views`: This directory contains the main view templates that represent individual pages of your application. 


## Setting up your own Template Engine
To set up your own template engine ensure you read the [Service Providers](/concept/providers) concept, find more about [Services](/basics/services) and how to [register a service](/basics/services#registering-a-service) in the documentation.

