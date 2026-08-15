---
title: "Supra CLI"
description: "The official CLI tool for managing your Supranim projects."
keywords: ["cli", "command line interface", "supra", "bootstrap", "project management"]
---

Supra CLI provides a set of commands to help you manage your Supranim projects efficiently. With Supra, you can easily create
new projects from starter kits and bundle static assets. This is pretty much the first version of the CLI, and I'm planning to add more features and commands in the future, so stay tuned for updates!

## Installation
Usually, Supra CLI is automatically installed when you install Supranim for the first time. However, if you need to install it, you can always do it using Nimble:

```
nimble install supra
```

Or, directly from GitHub using the following command:
```
nimble install https://github.com/supranim/supra
```

## Available Commands
Here are some of the available commands in Supra CLI. Type `supra -h` to see the full list of commands and options:
```text
CLI tool for bootstrapping Supranim applications
  (c) Supranim | MIT License  
  Build Version: 0.1.2

Development
  init <project:string>                       Create a new Supranim application
        --nocache:bool
        --skipconfig:bool
Asset Bundling
  bundle.assets <dir:string> <output:string>  Bundle static assets into the application
        --skip-prefix:bool
```

### The asset bundler
This is a simple command that takes a directory of static assets (like CSS, JS, images) and bundles them into your Supranim application as a virtual file system. This allows you to serve static files directly from your application.

🤯 This feature is used inside the [Booyaka Documentation Generator](https://github.com/openpeeps/booyaka) to bundle the necessary CSS, JS, SVG icons and other assets for the documentation website.
