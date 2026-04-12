---
title: "Query builder"
description: "Info about how to work with the query builder in Supranim."
keywords: ["query builder", "ozark", "database queries", "orm"]
---


## SQL Queries
Once you have defined your models, you can use them to interact with your database.
The Ozark ORM provides a powerful and flexible API for querying your database.

The API is designed to be intuitive and easy to use, it gives you compile-time safety,
so your queries are checked at compile time, this includes the types (data value,
table and column names) and also the SQL syntax.


### Insert Data
Inserting data into the database can be made using the `insert` macro
```nim
let id = Models.table(Users).insert({
  "email": "test@example.com"
  "password": "..."
}).execGet()
```

### Select Data
Select queries can be made using either `select` or `selectAll` macros. The select macro can be
combined with other macros like `where`, `orderBy`, `limit` and more.

```nim
let res: Collection[User] =
    Models.table(Users)
          .select(["name", "email"])
          .where("email", "test@example.com")
          .get()

assert not res.isEmpty
assert res.len == 1
```

Selecting all rows is not recommended for large tables, but it can be useful for small tables or for testing purposes.
```nim
let res: Collection[User] = Models.table(Users).selectAll().get()
assert not res.isEmpty
```

The returned result is always a `Collection[T]`, where `T` is the model type, even if you select
only one row, this is to keep the API consistent and avoid confusion.
