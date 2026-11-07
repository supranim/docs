---
title: "Templates"
description: "Render HTML views with the Tim template engine: layouts, partials and views."
tags: ["templates", "views", "rendering", "tim", "nimja", "template engine"]
---

## Tim Engine
[Tim Engine](https://github.com/tim-engine/tim) is an interpreted front-end engine with a built-in Virtual Machine, JIT (powered by [VanCode](https://github.com/openpeeps/vancode) + [pure Nim DynASM](https://github.com/nimbase/dynasm-nim)) and a beautiful DSL (Domain Specific Language) for creating templates. It provides a clean syntax for embedding dynamic data into your HTML views, making it easy to create reusable and maintainable templates. For more info, check dedicated documentation page: https://tim.openpeeps.dev

### Service Provider
The starter kits ship a ready-made Tim integration as a `Global` singleton service in [src/service/provider/tim.nim](https://github.com/supranim/starterkit/blob/main/src/service/provider/tim.nim). It wraps a single `TimEngine` instance and exposes it to every controller through the `render` template, you don't touch this file unless you want to extend it.

It is initialized in [src/app.nim#L42](https://github.com/supranim/starterkit/blob/main/src/app.nim#L42) alongside the other providers:

```nim
tim.init(
  App.config("tim.source").getStr,
  App.config("tim.output").getStr,
  supranim.basePath,
  global = %*{
    "system": {"isDevelopment": when not defined(release): true else: false},
    "date": {"year": now().year}
  }
)
```

`source`/`output` come from the `tim` section of the app config and point at the templates directory and its compiled cache; `global` is JSON data available to every template; `basePath` is the project root.

Rendering happens inside controllers with the `render` template from the service API:

```nim
import ../service/provider/[db, session, tim]

ctrl getHomepage:
  ## renders the home page
  render("index", local = &*{
    "isAuth": false
  })
```

`render(view, layout = "base", httpCode = Http200, local = nil)` renders the view inside the given layout and responds with `text/html`. On failure it logs the error and falls back to the `errors.5xx` view, or a plain `Internal Server Error` if that fails too. `getTimInstance()` returns the raw engine when you need it outside a controller.

### Template Files
By default, Tim Engine looks for template files in the `src/templates` directory of your project. Inside this directory you can organize your templates in in `layouts`, `partials` and `views` subdirectories:

- `layouts`: This directory contains layout templates that define the overall structure of your pages. Layouts typically include common elements like headers, footers, and navigation bars.
- `partials`: This directory contains partial templates that can be included in other templates. Useful for reusable components (forms, buttons), or any other HTML snippets that you want to use across multiple pages.
- `views`: This directory contains the main view templates that represent individual pages of your application. 

## Setting up your own Template Engine
To set up your own template engine, read the [Service Providers](/concept/providers) concept to learn how services are registered in your application.

## Alternative engines

### Built-in View
Core ships dependency-free view helpers in `supranim/support/view` for error pages and small responses, no engine setup required:

```nim
import supranim/support/view

# {{key}} is HTML-escaped, {{{key}}} is raw, unknown keys become ""
let body = renderTemplate("Hi {{name}}!", [("name", name)])

# full HTML5 document with an escaped title
respondPage("Hello", "<h1>Hi</h1>")
```

`escapeHtml`, `layout(title, bodyHtml, headExtra)` and `renderPage` compose pages manually; `respondPage(title, bodyHtml, code = Http200)` prepares the `text/html` response and blocks, mirroring the `json`/`respond` templates.

### Nimja Template Engine
[Nimja](https://github.com/enthus1ast/nimja) is a typed, compiled template engine inspired by Jinja2/Twig. Templates are transformed to Nim code at compile time, so you have the full power of Nim inside `{% %}` and `{{ }}`: control structures (`if`/`elif`/`else`, `when`, `case`, `for`, `while`), `{% extends %}`

```nim
# nimble install nimja
import nimja

type User = object
  name: string
  age: int

proc renderIndex(title: string, users: seq[User]): string =
  compileTemplateFile("index.nimja", baseDir = getScriptDir())
```

`compileTemplateFile` (or `compileTemplateStr`, with the `tmplf`/`tmpls` shorthands for inline calls) generates the body of your render proc, so template code sees the proc's variables directly. Pass `baseDir = getScriptDir()` whenever templates extend or import each other. Being a compiled engine, template changes require recompiling the app (use `nim c -f` if Nim misses a template edit); Nimja also ships experimental hot-reload via a shared library. See its [HCR docs](https://github.com/enthus1ast/nimja#automatic-recompilation--hot-code-reloading-hcr).
