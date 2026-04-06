---
title: "Configuration"
description: "Application configuration and settings."
keywords: ["configuration", "settings", "application", "environment variables", "config files"]
---

The configuration files are stored in the `config` directory of the Supranim project. There you can store your application settings, and other configuration options. The configuration files are in YAML format, which is easy to read and write.

## Environment Variables
Supranim also supports environment variables for configuration. You can set environment variables in a `.env.yml` file in the root of your project. This file should be in YAML format or in the `.env` format (key=value pairs). Environment variables set in this file will override any settings in the configuration files. 

### Autoloading Configuration
Supranim automatically loads configuration files from the `config` directory. You can organize your configuration files in subdirectories, and Supranim will load them all. For example, you can have a `config/database.yml` file for database settings and a `config/server.yml` file for server settings.

### Accessing Configuration in Code
You can access the configuration settings in your Nim code using the `config` module. For example, to access a field from the `database.yml` file, you can do the following:

```
configs("database.host").get()
```