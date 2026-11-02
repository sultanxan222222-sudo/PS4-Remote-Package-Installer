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


## Second-pass source fixes committed to this repository

The upstream text source tree has now been copied into `RPI/` (all 37 files listed by the upstream RPI directory). The original project Makefile and Visual Studio solution are also present.

The following changes are applied to the copied sources:

1. **Jailbreak helper safety (`RPI/main.c`)** — validate the module handle and all three `sceKernelDlsym` results before calling the resolved functions. Startup now fails cleanly if a required symbol is unavailable.
2. **HTTP range correctness (`RPI/http.c`)** — when a non-zero byte offset is requested, reject a server response of HTTP 200 instead of accepting it as the requested range. Reject short reads instead of reporting them as successful.
3. **HTTP error preservation (`RPI/http.c`)** — cleanup calls no longer overwrite the original request/callback error. This matters because otherwise a rejected Range response could be accidentally reported as successful after request cleanup.
4. **API request-body length (`RPI/server.c`)** — pass the actual body length to API handlers instead of `sizeof(data)`, which is only the size of a pointer. The POST body is copied and NUL-terminated before JSON parsing.

## Remaining build blockers / verification

- Binary runtime modules and package artwork have not yet been transferred into this repository.
- No OpenOrbis toolchain build has been run in this environment.
- No install/run test has been performed on the target PS4 Pro + GoldHEN 13.02.
- Therefore this repository contains a substantially imported and patched source tree, but it is **not yet a verified installable PKG**.
