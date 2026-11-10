---
title: "Database Models"
description: "Define your database models using the ORM provided by Supranim."
---

In Supranim you can define your database models and interact with
your database using the Ozark ORM. This allows you to work with your database in a more intuitive way, using Nim's type system and syntax.


## Defining Models
As specified in the [project structure](/concept/project-structure#model-directory), you can define your database models in the `src/model/` directory. All files in this directory are automatically loaded by Supranim at compile time, so you can define your models in any file within this directory. **It is recommended to organize your models in separate files for better maintainability.**

Here is an example of a simple `User` model defined in `src/model/user.nim`:
```nim
import supranim/model

newModel User:
  id {.pk.}: Serial
  email {.unique, notnull.}: Varchar(255)
  password {.notnull.}: Text
  created_at: TimestampTz
  updated_at: TimestampTz
```

## Using Models in Controllers
Using it inside a controller does not require any imports, as the model and its structure are automatically available to all controllers. You can use the model like this:
```
ctrl getUsers:
  ## `GET` handler for rendering a list of users
  withDBPool:
    let users = Models.table(User).selectAll().where("name", "John").getAll()
```

## Drivers: PostgreSQL, MySQL and SQLite
`supranim/model` is a facade over the Ozark ORM — `import supranim/model` gives you `newModel` plus the query, collection and migration layers. Pick one driver per binary:

```
import ozark/driver/psql   # PostgreSQL, default
import ozark/driver/mysql  # MySQL / MariaDB
import ozark/driver/sqlite # SQLite, file-based, no server
```

Or let `supranim/model/database` pick it from the build define (`-d:supraDBMainSqlite`, `-d:supraDBMainMysql`, else postgres) and initialize from the environment:

```
import supranim/model/database

initDatabaseFromEnv()  # poolSize = 10 by default
```

Recognized variables: `DB_TYPE` (`sqlite` | `postgres` with `postgresql`/`pgsql`/`pg` aliases | `mysql` with `mariadb`/`my` aliases), `DB_HOST` (default `localhost`), `DB_PORT` (defaults `5432` / `3306`), `DB_NAME`, `DB_USER`, `DB_PASS` and `DB_FILE` (sqlite path, `DB_NAME` alias, default `app.db`). A `DB_TYPE` that disagrees with the compiled driver raises `DbConfigError` instead of generating SQL for the wrong backend.

## Next
@database/queries.md

@database/collection.md