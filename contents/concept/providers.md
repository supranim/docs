---
title: "Service Providers"
description: "Manage your application's services and dependencies with the Service Provider system."
tags: ["service providers", "providers", "services"]
---

Service Providers are a powerful feature in Supranim that allows you to manage your application's services in a clean and organized way. They are responsible for **registering services**, and **bootstrapping your application**.

## Types of Service Providers
Supranim provides multiple types of service providers. A service provider can be registered as a **Singleton**, **Global**, **ThreadService**, **ThreadPoolService**, **WebService** or **UnixService** for standalone microservices and thread-based servers.

### Singleton Service Provider
A Singleton Service Provider is a service that is instantiated only once and shared across the entire application. It is useful for services that maintain state or need to be accessed globally, such as a database connection or a cache.

```nim
import supranim/core/services

initService Counter[Singleton]:
  description = "In-process hit counter"
  state do:
    type Counter = object
      hits: int
  api do:
    proc counter*(): ptr Counter =
      ## Return the singleton instance, creating it on first use
      getCounterInstance(
        proc(instance: ptr Counter) =
          {.gcsafe.}:
            instance[] = Counter(hits: 0)
      )

    proc hit*() =
      ## Record one hit
      inc counter()[].hits

    proc hits*(): int =
      ## Total hits recorded
      counter()[].hits
```

### Global Service Provider
A Global Service Provider is a namespace-style service without an instance type: only the `state`/`api` blocks are emitted, with no `get<Name>Instance` accessor. It is useful for grouping pure functions and constants under a service name.

```nim
import std/options
import pkg/mimedb
import supranim/core/services

initService Mime[Global]:
  description = "Shared MIME lookup helpers"
  state do:
    const DefaultMimeType = "application/octet-stream"
  api do:
    proc mimeType*(ext: string): string =
      ## Return the MIME type for `ext`, falling back to a default
      getMimeType(ext).get(DefaultMimeType)

    proc isKnownExtension*(ext: string): bool =
      ## True when `ext` exists in the MIME database
      isExtension(ext)

    proc isCompressibleType*(mimeType: string): bool =
      ## True when `mimeType` is marked as compressible
      isCompressible(getMimeInfo(mimeType))
```

### ThreadService Provider
A ThreadService runs user code in its own thread for as long as it wants and talks to the main application through the application-owned `Chan[ServiceMsg]`. It cannot be built standalone. (`ChannelService` is a deprecated alias of `ThreadService`.)

```nim
import std/strutils
import supranim/application
import supranim/core/services

initService Worker[ThreadService]:
  description = "Uppercase worker thread"
  queueSize = 32
  autoStart = false

  thread do:
    let ch = getWorkerChannel()
    while true:
      let msg = ch[].recv()
      if isServiceStop(msg):
        break
      ch[].send(ServiceMsg(id: msg.id, action: "done:" & msg.action,
        payload: msg.payload.toUpperAscii()))

  api do:
    proc submit*(app: Application, action, payload: string,
        id: int64 = 0): bool =
      ## Queue one job for the worker. Returns false when the queue is full.
      app.sendServiceMsg("Worker", action, payload, id)
```

When `autoStart` is false, the service does not start on its own. Start and stop it with the generated `start<Name>Service` and `stop<Name>Service` procs:

```nim
# generated function as `start<ServiceName>Service`
app().startWorkerService()

assert app().hasServiceChan("Worker")
assert app().submit("upper", "hello world", 1)

let reply = app().recvServiceMsg("Worker")
assert app().stopWorkerService()
```

The string name (`"Worker"`) is only used for channel lookup and messaging (`hasServiceChan`, `sendServiceMsg`, `recvServiceMsg`). Starting itself uses the typed proc.

### ThreadPoolService Provider
A ThreadPoolService owns a `TaskManager` (see [Background Tasks & Queues](/services/tasks)): N anonymous workers run submitted jobs while callbacks fire serialized on the pool dispatch thread, and a private scheduler thread drives delayed, repeating and wall-clock tasks. Jobs are submitted via the `api` handles, there is no worker body and no `routes`, `ws` or `thread do:` block. It cannot be built standalone.

```nim
import supranim/application
import supranim/core/services

initService Jobs[ThreadPoolService]:
  description = "CPU-bound job pool example"
  poolSize = 2
  autoStart = false

  shared do:
    type FibJob = object
      n: int

  api do:
    proc fib(n: int): int =
      ## Iterative Fibonacci, our CPU-bound demo job
      var a = 0
      var b = 1
      for _ in 2..n:
        let t = a + b
        a = b
        b = t
      b

    proc submitFib*(n: int, done: proc(res: int) {.closure.}): JobId =
      ## Run `fib(n)` off the calling thread. Returns a `JobId` for
      ## `cancelJob` (`JobId(0)` when the pool is down). `done` fires
      ## on the pool dispatch thread when the result is ready.
      getJobsPool().submit(
        job = proc(): int {.closure.} = fib(n),
        cb = done,
        onError = proc(err: ref CatchableError) {.closure.} =
          echo "job failed: ", err.msg)

    proc cancelFib*(id: JobId): bool =
      ## Drop a still-queued `fib` job: true means it will never run
      ## (silent). A running or finished job returns false and
      ## delivers normally.
      getJobsPool().cancelJob(id)
```

### WebService Provider
A WebService Provider is a standalone service that runs its own web server. It is useful for creating REST API microservices that need to run independently from the main application.

```nim
import supranim/microservice

initService Greeter[WebService]:
  description = "Hello microservice example"
  port = 8765

  routes do:
    get "/hello":
      req.respond(200, %*{"message": "hello from a standalone service"})

    get "/hello/{name:slug}":
      req.respond(200, %*{"message": "hello " & req.params["name"]})
```

A `WebService` can also run a standalone WebSocket server with a `ws do:` block instead of `routes do:`:

```nim
import supranim/microservice

initService Echo[WebService]:
  description = "WebSocket echo example"
  port = 8766

  ws do:
    wss.onOpen(proc(ws: WsConnection) {.closure.} =
      echo "ws client connected"
    )
    wss.onMessage(proc(ws: WsConnection, kind: WsFrameKind,
        data: openArray[byte]) {.closure.} =
      if kind == wsText:
        ws.sendText("echo: " & cast[string](data))
    )
    wss.onClose(proc(ws: WsConnection, code: int,
        reason: string) {.closure.} =
      echo "ws client left: ", code, " ", reason
    )
```

### UnixService Provider
A UnixService Provider is the same as a WebService Provider, but served over a Unix domain socket instead of TCP, which makes it ideal for fast local IPC. It supports `socketPath` and `socketMode` fields.

```nim
import supranim/microservice

initService LocalGreeter[UnixService]:
  description = "Local IPC microservice example"
  socketPath = "/tmp/supranim_example.sock"

  routes do:
    get "/ping":
      req.respond(200, %*{"pong": true})
```