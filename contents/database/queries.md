---
title: "Query builder"
description: "Use the Ozark query builder to insert, select, filter, update and delete rows with compile-time checked SQL."
keywords: ["query builder", "ozark", "database queries", "orm"]
---

## SQL Queries
Once you have defined your [models](/database/models), you can use them to interact with your database through the Ozark ORM. The query builder generates SQL at compile time, so table names, column names and value types are validated before your application compiles.

All queries run inside a database context: either `withDB` (one connection) or `withDBPool` (a connection from the pool):

```nim
withDB do:
  # execute queries here

withDBPool do:
  # execute queries here
```

The examples below use the `User` model from [Models](/database/models):

```nim
newModel User:
  id {.pk.}: Serial
  email {.unique, notnull.}: Varchar(255)
  password {.notnull.}: Text
  created_at: TimestampTz
```

### Create and drop tables
Tables are created automatically from the model definition:

```nim
withDB do:
  Models.table(User).prepareTable().exec()
  Models.table(User).dropTable(cascade = true).exec()
```

### Insert data
The `insert` macro takes a table constructor of `column: value` pairs. Use `execGet` to get the id of the inserted row, or `exec` to ignore it:

```nim
withDBPool do:
  let id = Models.table(User).insert({
    email: "test@example.com",
    password: "some-hash",
    created_at: $now(),
  }).execGet() # returns the id of the inserted row

  Models.table(User).insert({
    email: "another@example.com",
    password: "some-hash",
  }).exec()
```

### Select data
Select specific columns with `select`, or every column with `selectAll`:

```nim
withDBPool do:
  let res = Models.table(User)
                  .select(["email", "created_at"])
                  .getAll()

  let res = Models.table(User).selectAll().getAll()
```

### Filter with `where`
The `where` macro follows a `select`/`selectAll` clause. Chain conditions with `whereNot`, `orWhere` and `orWhereNot`:

```nim
withDBPool do:
  let res = Models.table(User)
                  .selectAll()
                  .where("email", "test@example.com")
                  .getAll()

  # NOT equal
  let res = Models.table(User)
                  .select("email")
                  .whereNot("email", "test@example.com")
                  .get()

  # OR conditions
  let res = Models.table(User)
                  .select("email")
                  .where("email", "a@example.com")
                  .orWhere("email", "b@example.com")
                  .get()

  let res = Models.table(User)
                  .select("email")
                  .whereNot("email", "a@example.com")
                  .orWhereNot("email", "b@example.com")
                  .get()
```

### Filter with `LIKE`
Ozark provides pattern-matching helpers for partial matches. `whereStartsLike` and `whereEndsLike` add the wildcard on the correct side automatically:

```nim
withDBPool do:
  # any position: %val%
  let res = Models.table(User)
                  .select("email")
                  .whereLike("email", "test")
                  .get()

  # prefix: val%
  let res = Models.table(User)
                  .select("email")
                  .whereStartsLike("email", "test")
                  .get()

  # suffix: %val
  let res = Models.table(User)
                  .select("email")
                  .whereEndsLike("email", ".com")
                  .get()

  # negative variants
  let res = Models.table(User)
                  .select("email")
                  .whereNotLike("email", "test")
                  .get()
```

### Filter with `IN`
Use `whereIn` and `whereNotIn` to match a list of values:

```nim
withDBPool do:
  let res = Models.table(User)
                  .select("email")
                  .whereIn("email", ["a@example.com", "b@example.com"])
                  .get()

  let res = Models.table(User)
                  .select("email")
                  .whereNotIn("email", ["a@example.com"])
                  .get()
```

### Ordering and limiting
Order the results with `orderDescBy`, and limit the number of rows with `limit`:

```nim
withDBPool do:
  let res = Models.table(User)
                  .selectAll()
                  .orderDescBy(["created_at"])
                  .getAll()

  let res = Models.table(User)
                  .selectAll()
                  .orderDescBy(["created_at"])
                  .limit(10)
                  .getAll()

  let res = Models.table(User)
                  .selectAll()
                  .where("email", "test@example.com")
                  .limit(1)
                  .get()
```

### Update data
The `update` macro sets columns, then `where` selects the rows to update:

```nim
withDBPool do:
  Models.table(User).update({
    email: "new@example.com",
  }).where("id", "1").exec()
```

### Delete data
`removeRow` builds a `DELETE FROM` statement. Combine it with `where` to delete a specific row; without a `where` clause it removes every row in the table:

```nim
withDBPool do:
  Models.table(User).removeRow().where("id", "1").exec()
```

### Raw SQL
When the query builder is not enough, `rawSQL` runs raw SQL with **parameter binding** to prevent SQL injection, while keeping compile-time validation. Use `getWith` to map results to a model:

```nim
withDBPool do:
  let res = Models.rawSQL(
    "SELECT * FROM users WHERE email = $1",
    "test@example.com"
  ).getWith(User)

  # raw SQL with subqueries
  let res = Models.rawSQL("""
SELECT
  users.*,
  (SELECT COUNT(*) FROM subscriptions WHERE subscriptions.user_id = users.id) AS subscriptions_count
FROM users ORDER BY users.id DESC LIMIT 20 OFFSET 0;
""").getWith(User)
```

### Result: `Collection[T]`
Query results are always a `Collection[T]`, where `T` is the model type, even when selecting a single row. This keeps the API consistent:

```nim
withDBPool do:
  let res = Models.table(User)
                  .selectAll()
                  .where("email", "test@example.com")
                  .get()

  assert not res.isEmpty
  assert res.len == 1
  assert res.get(0).email == "test@example.com"
  assert res.entries[0].email == "test@example.com"

  for user in res:
    echo user.email
```
