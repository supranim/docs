---
title: "Database Models"
description: "Define your database models using the ORM provided by Supranim."
---

In Supranim you can define your database models and interact with
your database using the Ozark ORM. This allows you to work with your database in a more intuitive way, using Nim's type system and syntax.


## Defining Models
As specified in the [project structure](/concept/project-structure#model-directory), you can define your database models in the `src/model/` directory. All files in this directory are automatically loaded by Supranim at compile time, so you can define your models in any file within this directory. **It is recommended to organize your models in separate files for better maintainability.**

Here is an example of a simple `User` model defined in `src/model/user.nim`:
```
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

## Next
@database/queries.md

@database/collection.md