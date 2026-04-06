---
title: "Query builder"
description: "Info about how to work with the query builder in Supranim."
keywords: ["query builder", "ozark", "database queries", "orm"]
---


## SQL Queries
Once you have defined your models, you can use them to interact with your database. The Ozark ORM provides a powerful and flexible API for querying your database.

The API is designed to be intuitive and easy to use, it gives you compile-time safety, so your queries are checked at compile time, this includes the types (data value, table and column names) and also the SQL syntax.


### Insert Data
```
withDBPool:
  let id = Models.table(Users).insert({
    "email": "test@example.com"
    "password": "..."
  }).execGet()
```