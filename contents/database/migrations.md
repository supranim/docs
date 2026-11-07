---
title: "Database Migrations"
description: "Version your database schema with Ozark migrations, re-exported through supranim/model."
keywords: ["database migrations", "ozark", "schema updates", "orm"]
---

Supranim re-exports the Ozark migration system through `supranim/model`, so it works with all three drivers (PostgreSQL, MySQL, SQLite). Migrations are Nim sources compiled as shared libraries and loaded at runtime from a folder. Bodies reuse the normal Ozark DSL with full compile-time validation.

## Writing a migration
Create one file per migration (ordered by the version in the filename) that imports your models:

```nim
# migrations/add_users_20260915120000.nim
import app/models
import ozark/migration

newMigration addUsers:
  ## Add users table with unique email
  up do:
    Models.table(Users).prepareTable().exec()
  down do:
    Models.table(Users).dropTable().exec()
```

## Applying migrations
On the host, import a driver and run them inside a pooled connection. Applied migrations are tracked in the `ozark_migrations` table (`compiledAt`/`runAt`):

```nim
import ozark/driver/sqlite # or psql / mysql
import ozark/migration

initOzarkDatabase("app.db")
withDBPool do:
  migrate("migrations")        # apply pending, ordered by filename version
  # rollback("migrations", 1)  # revert the last applied migration
  # for s in migrationStatus("migrations"): echo s.info.version, " ", s.applied
```

Migration bodies never touch a connection directly: `up`/`down` are expanded per driver and captured as SQL plus bound parameters, then the host executes them inside its own pooled transaction.
