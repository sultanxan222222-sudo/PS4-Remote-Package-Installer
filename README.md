# PS4 Remote Package Installer

Compatibility-focused Remote Package Installer for PS4.

## Target test environment
- Console: PS4 Pro
- Homebrew environment: GoldHEN
- System software: 13.02
- Sender: Windows PC on the same local network

## Upstream
This project is intended to start from the OpenOrbis SDK port maintained in [illusionyy/PS4RPI](https://github.com/illusionyy/PS4RPI), which credits [flatz/ps4_remote_pkg_installer](https://github.com/flatz/ps4_remote_pkg_installer) for the original implementation.

The upstream OOSDK port documents TCP port **12801** as its default listener port. Do not assume another port: verify the actual build/configuration on the console.

## Compatibility work plan

1. Import the complete upstream source and preserve its license and attribution.
2. Audit startup, network initialization, listener binding, API request parsing, package URL handling, and BGFT task creation.
3. Verify HTTP byte-range behavior end to end. The PS4 reads package metadata at offsets inside the PKG; the PC file server must correctly implement `Range: bytes=...`, return `206 Partial Content`, and provide a correct `Content-Range`. A server that always returns the whole file can cause package metadata parsing to fail.
4. Verify filenames and URLs containing spaces and non-ASCII characters.
5. Add clear startup diagnostics for the console IP, listening port, and network errors without exposing private data.
6. Build a reproducible PKG with the OpenOrbis toolchain.
7. Test on the exact target: PS4 Pro + GoldHEN + system software 13.02. Record successful connection, package metadata read, download, and installation separately.

## Important runtime note
The original Remote Package Installer documentation says the app must remain in the foreground while it receives the remote install command. After the task has been accepted and the download phase is underway, background behavior may differ. Keep the app open during connection and task registration.

## Current status

- [x] Repository created.
- [x] Target console environment recorded.
- [x] Upstream source tree imported (37 files under `RPI/`).
- [x] Bundled runtime PRX modules imported.
- [x] Startup symbol checks, HTTP Range handling, HTTP error propagation, and API request-body length fixes committed.
- [x] Import upstream package artwork and runtime modules required by the package.
- [x] Reproducible OpenOrbis PKG build (GitHub Actions run #17).
- [ ] Hardware test on PS4 Pro / GoldHEN 13.02.

**No working PKG or verified compatibility claim is published yet.** The complete upstream source and package inputs are committed with code fixes, but a successful toolchain build and console test are still outstanding.

## Build prerequisites
The upstream OOSDK project documents the OpenOrbis PS4 Toolchain and the `OO_PS4_TOOLCHAIN` environment variable. Its repository includes a Visual Studio solution and a Makefile; the exact build command must be confirmed against the imported source tree.


## Current source status

- Imported upstream RPI source files and build definitions.
- Added checks for jailbreak-helper symbol resolution.
- Added strict HTTP 206 validation for non-zero Range requests and short-read rejection.
- Preserved request errors while HTTP resources are cleaned up.
- Fixed API handler request-body length handling.
- Build and PS4 installation remain unverified.


## Automated build

A GitHub Actions build workflow is available at [build.yml](.github/workflows/build.yml). It is configured to build with OpenOrbis Toolchain v0.5.4 and publish the generated PKG as an Actions artifact. Successful build: [GitHub Actions run #17](https://github.com/sultanxan222222-sudo/PS4-Remote-Package-Installer/actions/runs/41532908036). The generated PKG is available as an Actions artifact. Hardware installation and runtime testing are still outstanding.
