---
title: "Background Tasks & Queues"
description: "Run ephemeral background tasks on a ThreadPoolService TaskManager, and durable queue jobs with retries, backoff and a failed-jobs table."
keywords: ["background tasks", "task scheduler", "job queue", "cron jobs", "scheduled tasks", "queue worker"]
---

## About
Supranim has two job systems, and they solve different problems:

- **Tasks** are ephemeral, in-process concurrency: fire-and-forget work, delayed and repeating timers, wall-clock schedules and cancellation. Nothing survives a restart. They run on a `TaskManager` owned by a `ThreadPoolService` provider.
- **Queues** are durable, async work: `dispatch*` stores a row in `queue_jobs`, a worker executes it later, with retries, backoff, delayed availability and a `failed_queue_jobs` table. Rows survive restarts and crashes.

### Example usage
- Use tasks for **cache refreshes**, **timeouts**, **heartbeats** and "_run this at 09:00_". 
- Use queues for **welcome emails**, **receipts** and **webhooks**, anything that must not be lost on deploy.

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">
    The task engine is also available as a standalone Nimble package (`supranim_tasks`, framework-agnostic: only powpow plus the standard library) for any Nim project.
  </div>
</div>

## The pool service
A `ThreadPoolService` provider owns a `TaskManager`: N worker threads plus a private scheduler thread. Define one outside the main module (pool services cannot be built standalone):

```nim
import supranim/application
import supranim/core/services

initService Jobs[ThreadPoolService]:
  description = "CPU-bound job pool"
  poolSize = 2
  autoStart = false

  api do:
    proc submitFib*(n: int, done: proc(res: int) {.closure.}): JobId =
      getJobsPool().submit(
        job = proc(): int {.closure.} = fib(n),
        cb = done)
```

The provider generates `getJobsPool(): TaskManager`, `startJobsService(app, poolSize = 4)`, `stopJobsService(app)`, plus `getJobsRawPool()` (the underlying powpow pool, rarely needed), `isJobsServiceRunning()` and `haltJobsService(app, delayMs)` — the only stop that is safe from inside a job or callback, since `stop` joins pool threads.

## Immediate, delayed and repeating tasks

```nim
let app = appInstance()
assert startJobsService(app)

# Immediate: runs on a worker, callback fires serialized on the
# pool dispatch thread (never on the caller thread).
let id = getJobsPool().submit(
  proc(): string = "hello",
  proc(res: string) = echo "got: ", res)

# Cancel a still-queued job: true means it will never run (silent).
# Running jobs cannot be preempted: false, and they deliver normally.
if getJobsPool().cancelJob(id):
  echo "was still queued"

# Delayed one-shot and repeating interval timers.
let once = getJobsPool().submitDelayed(500,
  proc(): int = 40 + 2,
  proc(res: int) = echo res)
let every = getJobsPool().submitRepeating(1000,
  proc(): int = tick(),
  proc(res: int) = echo res, name = "ticker")

assert stopJobsService(app)
```

`submit` returns a `JobId` (`JobId(0)` means rejected: stopping or closed). Delayed and repeating tasks return a `TimerId` and accept an optional `name = "..."`.

## Wall-clock scheduling
Daily and weekly tasks chain one-shots recomputed from local time after every fire, so DST shifts land on one 23h/25h day instead of drifting:

```nim
import std/times

# Once at a DateTime (past times never fire: they stay tracked
# as taskInactive, name reserved, no warning).
getJobsPool().scheduleAt(
  (getTime() + initDuration(seconds = 30)).local(),
  proc(): int = 1, proc(res: int) = echo "half a minute",
  name = "once")

# Every day / week at a local wall-clock time.
getJobsPool().scheduleDaily(9, 0, 0,
  proc(): int = 1, proc(res: int) = echo "morning",
  name = "digest")
```

Inspect and stop timers by id or name. Cancelling keeps the name reserved (a duplicate name raises); removing frees it:

```nim
getJobsPool().cancelTask("ticker")   # stops firing, name reserved
getJobsPool().removeTask("ticker")   # cancels + frees the name
assert getJobsPool().taskStatus("digest") == taskArmed
assert getJobsPool().hasTask("digest")
```

`taskStatus` reports `taskArmed`, `taskCancelled`, `taskInactive` or `taskUnknown`. Timer control operations take effect at the next scheduler heartbeat (~10ms); `cancelJob` is synchronous. For chained daily/weekly tasks, prefer cancelling by name: a stale id still resolves to the live occurrence.

## Queue jobs
Define typed jobs with the `job` macro in `src/service/event/queue` (auto-discovered like event listeners). This generates a `<Name>Payload` object, a typed `handle<Name>` proc, a `dispatch<Name>` proc that inserts a row, and a `QueueRegistry` entry for the worker:

```nim
import pkg/ozark/driver/sqlite
import pkg/supranim/queue/model
import pkg/supranim/service/queue

job SendWelcomeEmail:
  queue = "emails"
  tries = 3
  backoff = 60
  timeout = 120

  payload:
    userId: int
    email: string

  handle(payload):
    Mailer.sendWelcome(payload.userId, payload.email)
```

Dispatch from anywhere (it owns its `withDBPool` — never call it from inside another pool block). `delay` postpones availability in seconds, `onQueue` overrides the queue:

```nim
dispatchSendWelcomeEmail(SendWelcomeEmailPayload(userId: 7, email: "a@b.c"))
dispatchSendWelcomeEmail(SendWelcomeEmailPayload(userId: 8, email: "d@e.f"), delay = 3600)
```

Run the worker on any `TaskManager` with the `service/queue_work` bridge. It polls `queue_jobs` every `intervalMs`, claims up to `batch` due rows (`available_at <= now`, ordered by `priority, available_at`), and runs each `handle` on the pool dispatch thread:

```nim
import pkg/supranim/service/queue_work

startQueueWorker(getJobsPool(), queueName = "emails",
  intervalMs = 1000, batch = 5)
# ...
stopQueueWorker(getJobsPool())
```

Settling is per row: success deletes it, failure re-queues it (`now + backoff`, attempts bumped) until `tries` runs out, then moves it to `failed_queue_jobs`. Unknown job names fail the row instead of crashing. Delivery is at-least-once with a read-only claim: keep one worker per queue and `intervalMs` comfortably above handle time. On SQLite, open the database in WAL mode so the polling reads never block the settling writes.

## Threading and lifecycle rules
- Lock shared state inside `cb`/`onError`/`handle` — they run on the pool dispatch thread, never on the caller thread. Never touch a request object there.
- Job closures must not capture `ref` objects across threads (values, strings, locks, atomics and raw pointers only).
- `stop` drains queued jobs gracefully, `shutdown` discards them; both block and must run outside pool jobs and callbacks — from inside one, call `halt` instead. `close` finishes teardown on the creating thread and is idempotent.

## Standalone usage
Use `supranim_tasks` directly in any Nim project (`requires "supranim_tasks >= 0.2.0"`, Nim 2.2.10+, threads on). The README of the [tasks package](https://github.com/supranim/tasks) documents the full API with runnable examples.

### API Reference
The API reference for the tasks engine: https://supranim.github.io/tasks

## Related
@services/events.md
@database/queries.md
