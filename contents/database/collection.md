---
title: "Collection"
description: "Work with query results through the ozark Collection wrapper: length, first, get, contains and iteration."
keywords: ["data collection", "ozark", "database collection", "orm"]
---

## What is a Collection?
`Collection[T]` is the type returned by the Ozark query builder for every `get`/`getAll` call. It wraps a sequence of model instances (`entries`) and provides helpers for inspecting and iterating the result set. `T` is the model type.

```nim
withDBPool do:
  let users = Models.table(User).selectAll().getAll()
  echo users.len            # number of rows
  echo users.isEmpty        # true if no rows
```

## The `entries` field
The raw sequence of items, accessible directly:

```nim
let firstUser = users.entries[0]
```

## Accessing items
Use `get(index)`, the `[]` operator, or `first` to access an item by position:

```nim
let first = users.first()          # first item (index 0)
let second = users.get(1)          # item at index 1
let third = users[2]               # item at index 2
```

`first` and `get` index into `entries` directly, so passing an out-of-range index raises an `IndexDefect`.

## Getting all items
`getAll` returns the underlying sequence, useful when you need a `seq[T]`:

```nim
let all: lent seq[User] = users.getAll()
```

## Checking contents
`contains(key, val)` returns `true` when any item has the given field set to the given value:

```nim
if users.contains("email", "test@example.com"):
  echo "found"
```

## Iterating
`Collection[T]` supports `for` loops via the `items` iterator, and `mitems` when you need to modify items in place:

```nim
for user in users:
  echo user.email

for user in users.mitems:
  user.name = "updated"
```

`mapIt` is also available for transforming the collection (exported by Ozark):

```nim
let emails = users.mapIt(it.email)
```
