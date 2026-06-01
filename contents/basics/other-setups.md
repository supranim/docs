---
title: "Other Setups & Low-level APIs"
description: "A quick overview of core networking setups and low-level APIs available in Supranim for building custom web servers, TCP/UDP servers, and WebSocket servers."
keywords: ["http", "server", "websocket", "tcp", "udp", "basic", "setup"]
---

## About
In some cases, you may want to build a custom HTTP server without using the full features of the Supranim framework. Supranim expose core networking APIs that allow you to create HTTP servers, TCP/UDP and WebSocket servers for various use cases

## Simple HTTP Server Example
Creating a simple HTTP server looks like this:

```nim
import std/osproc
import pkg/supranim/network/webserver

var server = newWebServer()

proc onRequest(req: var webserver.Request) =
  req.resp(Http200, "All cheese is good cheese")

server.start(onRequest, startupCallback = nil, threads = countProcessors())
```

## WebSocket Upgrade Example
You can tell Supranim to upgrade an incoming HTTP request to a **WebSocket connection** if it matches a certain route. For example:
```nim
server.registerCallback("/ws",
  proc (req: ptr evhttp_request, arg: pointer) {.cdecl.} =
    discard websocketUpgrade(req, onOpenCallback, nil, onClose, onError)
  )
```

## Benchmarking
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