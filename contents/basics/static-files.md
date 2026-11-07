---
title: "Static Files"
description: "Serve static assets with traversal protection, ETags, conditional requests and byte ranges."
keywords: ["static files", "assets", "etag", "cache", "range", "download"]
---

## Serving files
`supranim/core/fileserver` serves embedded assets (`service/assets`) and disk files (MIME types via `mimedb`). All disk procs guard against directory traversal (`..` segments are treated as not-found, never reaching the filesystem) and set `ETag`, `Last-Modified`, `Cache-Control: public, max-age=86400` and `Accept-Ranges: bytes`:

```nim
import supranim/core/fileserver

# inside a route handler; basePath keeps files rooted to one directory
if req.sendAssets(publicDir, req.getUriPath(), res.getHeaders()):
  discard
else:
  req.resp(Http404, "not found")

# fixed path, or prompt a download instead
req.sendAssetFile(publicDir / "app.css", res.getHeaders())
req.sendDownloadable(storageDir / "report.pdf", res.getHeaders())
```

Conditional requests are honored automatically: a matching `If-None-Match` answers `304` without resending the body. Valid `Range` requests stream a `206` partial response; unsatisfiable ranges answer `416` with `Content-Range: bytes */N`.

## Related
@basics/templates.md
