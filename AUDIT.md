# Initial source audit

Upstream reviewed: https://github.com/illusionyy/PS4RPI (branch `feat`).

## Confirmed project layout

- `RPI/main.c` — startup, module loading, network setup, server startup.
- `RPI/net.c` — PS4 network initialization and IPv4 lookup.
- `RPI/http.c` — HTTP/SSL client used to read remote PKG metadata and data.
- `RPI/installer.c` — BGFT and AppInstUtil integration.
- `RPI/server.c` — HTTP API and static file handling.
- `sce_module/libjbc.prx` — bundled jailbreak helper loaded by the application.
- `Makefile` and `RPI/build.bat` — Linux/macOS and Windows build paths.

## Findings from the first pass

1. `main.c` hardcodes TCP port `12801`.
2. `main.c` loads `/app0/sce_module/libjbc.prx`, then resolves `jbc_get_cred`, `jbc_jailbreak_cred`, and `jbc_set_cred` with `sceKernelDlsym`.
3. The return values from those three symbol lookups are logged but not validated before the function pointers are called. This is a concrete crash risk if the PRX loads but a symbol is unavailable or resolution fails. The next code change should validate the module handle and every symbol pointer, and fail cleanly rather than calling a null pointer.
4. Startup loads internal system, network, HTTP, SSL, AppInstUtil, BGFT, and NP modules before starting the listener.
5. The network initialization and HTTP download code need a separate detailed review for error handling, partial reads, range response validation, and cleanup paths.
6. The repository contains the helper PRX; this is not an external file the user needs to supply.

## Compatibility status

This source has not yet been built with a pinned OpenOrbis toolchain or tested on PS4 Pro + GoldHEN 13.02. No compatibility claim is made until that test is completed.
