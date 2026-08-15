---
title: "Other Setups & Low-level APIs"
description: "Build custom HTTP and WebSocket servers with the low-level APIs exposed by Supranim's powpow backend."
keywords: ["http", "server", "websocket", "tcp", "udp", "basic", "setup"]
---

## About
In some cases, you may want to build a custom HTTP server without using the full features of the Supranim framework. Supranim exposes core networking APIs, built on powpow, that allow you to create HTTP servers, TCP/UDP and WebSocket servers for various use cases.

## Examples using the powpow backend
You can skip the Supranim MVC structure and use the low-level API directly.

### A simple HTTP server
Creating a simple HTTP server looks like this:

```nim
import std/cpuinfo
import supranim/network/webserver

var server = newWebServer()

proc onRequest(req: var webserver.Request) =
  req.send(Http200, "All cheese is good cheese")

server.start(onRequest, startupCallback = nil, threads = countProcessors())
```

The `Request` type exposes `send(code, body)`, `sendFile`, `sendChunk` and streaming helpers for low-level responses.

### WebSocket upgrade example
You can tell Supranim to upgrade an incoming HTTP request to a **WebSocket connection** if it matches a certain route. Low-level callbacks registered with `registerCallback` receive the raw powpow `HttpRequest`/`HttpResponse` pointers:

```nim
import pkg/powpow as pw
import supranim/network/webserver

server.registerCallback("/ws",
  proc (req, arg: pointer) {.cdecl, gcsafe.} =
    let req = cast[pw.HttpRequest](req)
    let res = cast[pw.HttpResponse](arg)
    discard pw.websocketUpgrade(res, req,
      onOpen = proc(ws: pw.WsConnection) = echo "open!",
      onMessage = proc(ws: pw.WsConnection, kind: pw.WsFrameKind, data: openArray[byte]) =
        ws.sendText(cast[string](@data)),
    )
  )
```

### Benchmarks Supranim + powpow
Here you can find some stupid and unrealistic benchmarks for the above server setup:
```
Running 10s test @ http://127.0.0.1:8080
  12 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   768.38us    2.88ms  81.18ms   98.72%
    Req/Sec    15.38k     1.50k   23.09k    90.00%
  1851525 requests in 10.10s, 257.80MB read
Requests/sec: 183319.31
Transfer/sec:     25.52MB
```

Sure, passing `-H "Connection: close"` to wrk gives more realistic numbers:
```
Running 10s test @ http://127.0.0.1:8080
  12 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.35ms  520.77us   3.69ms   59.17%
    Req/Sec     1.51k   604.88     2.18k    77.06%
  16452 requests in 10.10s, 2.59MB read
Requests/sec:   1628.31
Transfer/sec:    262.37KB
```
