---
title: "Install Supranim"
description: "How to install Supranim and the Supra CLI."
---

<div class="alert alert-info rounded-4" role="alert">
  <div class="alert-content">
    Supranim requires Nim version 2.2.10 or higher. Make sure you have the latest version of Nim installed to use Supranim.
  </div>
</div>

## Installing Supranim
It's easy to get started with Supranim. You can install it with either of the two package managers for Nim development: **Nimble**, which ships with Nim, or [**Clue**, a DFS package manager alternative made by OpenPeeps](https://github.com/openpeeps/clue). If you don't have Nim installed, you can get it from the official website: [https://nim-lang.org/install.html](https://nim-lang.org/install.html).

Both flows below install the latest version of the **Supranim** framework and **Supra**, the CLI app used to create and **manage Supranim projects**.

## Installing with Nimble

```bash
nimble install supra supranim
```

Nimble places the `supra` binary in `~/.nimble/bin`, so make sure that directory is on your `PATH`.

## Installing with Clue
[Clue](https://github.com/openpeeps/clue) is an alternative package manager for Nim development with cached version discovery, transitive dependency resolution and per-version toolchains. Install it with Nimble:

```bash
nimble install clue
```

Clue installs binaries to `~/.clue/bin`, so add that directory to your `PATH` once. Then install the Supranim library and the Supra CLI:

```bash
clue install supranim
clue install supra --build
```

Note the `--build` flag on the second command: unlike Nimble, Clue never compiles a package's binaries implicitly, so `--build` is what produces the `supra` executable in `~/.clue/bin`.

### Dependencies
Supranim relies on several Nim packages to provide its features. Most of them are maintained [@supranim](https://github.com/supranim/supranim) and [@openpeeps](https://github.com/openpeeps) organizations and are Nim-only:

- **[kapsis](https://github.com/openpeeps/kapsis)**: CLI command parsing used by `App.cli`.
- **[powpow](https://github.com/openpeeps/powpow)**: HTTP/1.1 and WebSocket server (the Supranim backend).
- **[ozark](https://github.com/openpeeps/ozark)**: macro-based ORM with a type-safe query builder (PostgreSQL).
- **[emitter](https://github.com/supranim/emitter)**: event emitter powering the events service.
- **[nimcypher](https://github.com/nimbase/nimcypher)**: cryptography for `supranim/support/auth`: a pure-Nim port of Monocypher (X25519, Ed25519, XChaCha20-Poly1305, Argon2id).
- **[tim](https://github.com/openpeeps/tim)**: template engine.
- **[boogie](https://github.com/openpeeps/boogie)**: WAL-based KV and RDBMS stores used by the Cache service.
- Other utilities: `flysystem` (filesystem), `twofa` (QR/2FA), `openparser` (JSON/YAML/regex), `threading`, `flatty`, `jsony`, `mimedb`, `checksums`, `semver`.

For the complete list, see the [Nimble file](https://github.com/supranim/supranim) in the Supranim repository. Depending on the project type you choose when creating a new project, some additional dependencies may be required. 🔥 [Check the Starter Kits documentation](/starter-kits) for more details on the specific dependencies for each project type.

### No C build dependencies
The HTTP server (powpow) and the cryptography layer (nimcypher) are implemented entirely in Nim, so Supranim itself has **no C library build dependencies**. No additional C toolchain or libraries are required to compile a Supranim application.

The only exception is the database layer: the PostgreSQL/SQLite drivers

## Next steps
@starter-kits.md

@dependencies.md