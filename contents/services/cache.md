---
title: "Cache"
description: "Durable cache service backed by Boogie KV store with WAL, TTL and bucket isolation."
keywords: ["cache", "boogie", "kv store", "wal", "ttl"]
---

## Overview
The Cache service is a `WebService` provider at `src/supranim/service/cacheable.nim`.
It exposes an HTTP API over named buckets and stores entries in a single
Boogie `KvStore` at `storage/cache` (WAL enabled, checkpoint every 100 ops,
flush every 1000 ops). Entries are `flatty` serialized `CacheEntry` values
with optional `expiresAt` timestamps and are lazily expired on read.

## Routes

| Method | Path | Description |
|---|---|---|
| POST | `/storage/{bucket:slug}` | Create a bucket. Returns `201` or `409` if it exists |
| PUT | `/storage/{bucket:slug}/cache/{key:slug}` | Store entry. Body is the value. Supports `?ttl=60` or header `x-cache-ttl` for expiry in seconds |
| GET | `/storage/{bucket:slug}/cache/{key:slug}` | Retrieve entry. Returns `404` if missing or expired |
| PATCH | `/storage/{bucket:slug}/cache/{key:slug}` | Update entry value |
| DELETE | `/storage/{bucket:slug}/cache/{key:slug}` | Delete entry |
| POST | `/storage/{bucket:slug}/flush` | Delete all entries in the bucket |
| GET | `/storage` | List buckets with `length` and `created_at` |

TTL example:

```
PUT /storage/sessions/cache/abc123?ttl=3600
Content-Type: text/plain
hello world
```

or with header:

```
x-cache-ttl: 3600
```

## Implementation details
- Composite keys `bucket:key` and marker keys `__bucket:<name>` are used inside one `KvStore`.
- `GET /storage` scans `pairsUnordered`, groups by bucket prefix, and skips expired entries.
- Writes are synchronous for visibility and batched for WAL durability via Boogie.
- Expiry is checked on `withEntry`; expired keys are deleted and return `404`.

For the Boogie KV API see the [Boogie docs](https://github.com/openpeeps/boogie) and
`pkg/boogie/stores/kv`.

## Related
The Cache service is a standalone microservice. Your application talks to it
over HTTP via the generated client. See [@service providers](/concept/providers)
and [@boogie KV store](https://github.com/openpeeps/boogie) for more.
