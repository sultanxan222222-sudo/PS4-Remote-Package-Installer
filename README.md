# PS4 Remote Package Installer

Updated and compatibility-focused Remote Package Installer for PS4.

## Target
- PS4 Pro
- GoldHEN on system software 13.02
- Windows PC sender connecting to the console over the local network

## Project goals
- Preserve the original Remote Package Installer protocol and proven behavior.
- Verify compatibility with the target PS4 environment before claiming support.
- Keep the console-side listener stable and report useful connection/install errors.
- Provide reproducible builds and release PKG artifacts when build infrastructure is available.

## Upstream
Based on the OpenOrbis SDK port of `ps4_remote_pkg_installer` by illusionyy, which credits flatz for the original work.

Upstream: https://github.com/illusionyy/PS4RPI
Original project: https://github.com/flatz/ps4_remote_pkg_installer

## Status
Repository initialized. Source import, code review, build, and PS4 hardware testing are still pending. No compatibility claim is made until the resulting PKG is built and tested on the target console.
