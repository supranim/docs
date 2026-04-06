---
title: "Database Migrations"
description: "Run database migrations to keep your database schema up to date with your models."
keywords: ["database migrations", "ozark", "schema updates", "orm"]
---

Currently, Supranim does not have a built-in migration system. However, you can use the Ozark ORM to manage your database schema and keep it in sync with your models.

## The plan
There are plans to implement a migration system as part of the Supra CLI in the future. Basically, migrations will be compiled as shared libraries, and the CLI will be able to load and execute these libraries to apply or revert schema changes to the database.

This approach allows for a flexible and powerful migration system without losing the benefits of compile-time safety.

When executing a migration, the CLI will set the environment of the webserver to a special "migration mode", which will allow the migration code to run while the webserver is running in read-only mode, preventing any changes to the database from the webserver while the migration is running. This ensures that the database schema is updated safely without any conflicts or issues.