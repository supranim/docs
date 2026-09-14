---
title: "Other Setups & Low-level APIs"
description: "Build custom HTTP and WebSocket servers with the low-level APIs exposed by Supranim's powpow backend."
keywords: ["http", "server", "websocket", "tcp", "udp", "basic", "setup"]
---

Supranim ships with an idiomatic MVC structure built around routing, controllers, middleware and templates, but you are not required to use it that way. For custom web apps, microservices or lightweight prototypes that follow a non-idiomatic setup, you can skip the framework layer and work directly with the low-level networking APIs. Built on powpow, these APIs let you create HTTP servers as well as TCP/UDP and WebSocket servers for various use cases.

## Examples using the powpow backend
You can skip the Supranim MVC structure and use the low-level API directly.

### A simple HTTP server
Creating a simple HTTP server looks like this:

```nim
import supranim/network/webserver

var server = newWebServer()

proc onRequest(req: var webserver.Request) =
  req.send(Http200, "All cheese is good cheese")

server.start(onRequest, startupCallback = nil, threads = 2)
```

The `Request` type exposes `send(code, body)`, `sendFile`, `sendChunk` and streaming helpers for low-level responses.


### Adding Router
If you need routing without the `routes` macro DSL, use the runtime API from `supranim/core/router`. Create a router with `newHttpRouter`, register handlers with `registerRoute`, then dispatch inside `onRequest` with `checkExists`:

```nim
import std/[httpcore, tables]
import supranim/core/[router, request, response]
import supranim/network/webserver

proc helloHandler(req: var Request, res: var Response) {.nimcall, gcsafe.} =
  res.setBody("hello world")

proc usersHandler(req: var Request, res: var Response) {.nimcall, gcsafe.} =
  res.setBody("id=" & req.routeParams.getOrDefault("id"))

var router = newHttpRouter()
router.registerRoute("/hello", HttpGet, helloHandler)
router.registerRoute("/users/{id:id}", HttpGet, usersHandler)

proc onRequest(req: var Request) {.gcsafe.} =
  var res = Response(headers: newHttpHeaders())
  let rc = router.checkExists(req.getUriPath(), req.getHttpMethod())
  if rc.exists:
    req.routeParams = rc.params
    rc.route.callback(req, res)
    if not req.responseSent:
      req.resp(res.getCode(), res.getBody(), res.getHeaders())
  else:
    req.resp(Http404, "not found")

var server = newWebServer()
server.start(onRequest)
```

Each handler uses the `proc(req: var Request, res: var Response) {.nimcall, gcsafe.}` signature. Use `registerRoute` with `middlewares = @[...]` or `afterwares = @[...]` when needed, and resolve them with `resolveMiddleware` and `resolveAfterware` before and after the callback.


### WebSocket upgrade example
You can tell Supranim to upgrade an incoming HTTP request to a **WebSocket connection** if it matches a certain route. Low-level callbacks registered with `registerCallback` receive the powpow `HttpRequest` and `HttpResponse` directly:

```nim
import pkg/powpow as pw
import supranim/network/webserver

server.registerCallback("/ws",
  proc (req: pw.HttpRequest, res: pw.HttpResponse) {.gcsafe.} =
    discard pw.websocketUpgrade(res, req,
      onOpen = proc(ws: pw.WsConnection) = echo "open!",
      onMessage = proc(ws: pw.WsConnection, kind: pw.WsFrameKind, data: openArray[byte]) =
        ws.sendText(cast[string](@data)),
    )
  )
```
