---
title: "Backends"
description: "The HTTP and WebSocket server backends used by Supranim."
keywords: ["http", "server", "websocket", "tcp", "udp", "backend"]
---

Supranim's HTTP server and WebSocket support are built on **[powpow](https://github.com/openpeeps/powpow)**, a high-performance event notification library written entirely in Nim. It provides TCP/UDP, HTTP/1.1 and WebSocket server capabilities and is the only backend shipped with Supranim.

The web server lives in `supranim/network/webserver` and the WebSocket support in `supranim/network/websocket`, both re-exported through `import supranim`.

For more information related to low-level **UDP**, **TCP** and **WebSocket** sockets, check the [PowPow package on GitHub](https://github.com/openpeeps/powpow), and the [PowPow API reference](https://openpeeps.github.io/powpow/).

## Related
@basics/other-setups.md
