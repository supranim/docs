---
title: "WebSockets"
description: "Real-time communication with rooms, broadcast and session-aware upgrades."
keywords: ["websocket", "rooms", "broadcast", "real-time", "pub-sub"]
---

## Framework layer
`supranim/network/websocket` re-exports the powpow primitives (`WsConnection`, `sendText`/`sendBinary`, `websocketUpgrade`). The framework layer lives in `supranim/support/ws`: named rooms (pub-sub) on a thread-safe hub, an authenticated-upgrade helper and session-aware middleware. Standalone `WebService` apps keep using `ws do:` with `wss.onOpen`/`wss.onMessage`/`wss.onClose` (see `examples/services_ws.nim`); the hub works there too.

## Rooms and broadcast
```nim
import supranim/support/ws

# anywhere (route handlers, services, timers):
defaultHub().joinRoom("lobby", ws)
defaultHub().leaveRoom("lobby", ws)
defaultHub().leaveAllRooms(ws)          # call from onClose
defaultHub().broadcast("lobby", "hi")  # skips closed sockets
defaultHub().roomCount("lobby")
defaultHub().rooms()
```

`broadcast` copies members under a read lock and sends outside it — powpow's `sendText` is thread-safe and skips non-connected sockets, so broadcasting to a closed or `nil` connection is a safe no-op. Use `roomPeers(room, exclude = ws)` when you need the member list for custom fan-out, and `newWsHub()` when you want an isolated hub (e.g. tests).

## Authenticated app routes
Protect a `router.ws` route with the session middleware, then upgrade in the handler:

```nim
import supranim/support/ws

router.ws("/chat/{room:slug}", chatHandler,
  middlewares = @[requireWsSession])

proc chatHandler(req: var Request, res: var Response) {.nimcall, gcsafe.} =
  let room = req.routeParams.getOrDefault("room", "lobby")
  discard req.upgradeWs(res,
    onOpen = proc(ws: WsConnection) =
      defaultHub().joinRoom(room, ws),
    onMessage = proc(ws: WsConnection, kind: WsFrameKind,
                     data: openArray[byte]) =
      if kind == wsText:
        defaultHub().broadcast(room, cast[string](@data), exclude = ws),
    onClose = proc(ws: WsConnection, code: int, reason: string) =
      defaultHub().leaveAllRooms(ws))
```

`requireWsSession` lets the upgrade proceed only with a valid, non-expired session from the core in-memory store (`support/session`); otherwise the router answers `403`. For DB-backed users, write a similar middleware around `supranim-session`'s `isAuthenticated`. See `examples/ws_rooms.nim` for a complete app.

## Related
@basics/backends.md
