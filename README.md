# MagicRoot

English | [简体中文](README.zh-CN.md)

Android application and native sources for a temporary, reboot-cleared root
workflow on authorized devices. The app does not unlock the bootloader, does
not modify partitions, and does not create persistent root.

> Authorized testing only. The workflow loads unsigned kernel modules,
> temporarily changes SELinux state, and may panic or reboot the device. Use
> only on devices you own or are explicitly authorized to test.

## Features

- Two isolated chains: CVE-2026-64560 (staged pipeline) and CVE-2026-98260
  (UAF/rootd).
- Multiple manager backends: KernelSU, BakaSU, FinaSU, SukiSU Ultra,
  KernelSU-Next, SukiSU Nova and MapleSU, switchable in the UI.
- Runtime LKO patching: per-kernel symbol table rewrites undefined symbols to
  `SHN_ABS` and fixes vermagic.
- Manager LKO extraction from an installed manager APK.
- Adaptive 6.18 profile generation from a local `boot.img` (same `uname -r`).
- STP parallel-space cleanup and an in-place LSPosed hotfix.

## Layout

```
main/compose-app/   Compose UI and controller
main/app/           device scripts, assets, native code, build.ps1
binary-sources/     native payload sources, per-build headers
tools/              build, audit, packaging and test scripts
LICENSES/           third-party license texts
docs/               architecture, build and release notes
```

Prebuilt kernel modules and payload binaries are **not** included in this
source snapshot; rebuild them from `binary-sources/` or supply your own.

## Build

Release build (Windows, JDK 25 + Android SDK Platform 37; requires the full
runtime asset set):

```powershell
powershell -ExecutionPolicy Bypass -File .\main\app\build.ps1 `
  -Sdk <sdk> -Jdk <jdk25> `
  -Keystore <release.keystore> `
  -StorePass <storePass> -KeyAlias magicroot -KeyPass <keyPass>
```

Static checks (require the full runtime asset set):

```sh
bash tools/check.sh
```

The Gradle project is under `main/compose-app/`; the app assets are staged
from `main/app/`. See `docs/build.md`.

## Workflow

1. A shell/Shizuku path deploys the selected assets to
   `/data/local/tmp/magic`.
2. Assets are filtered by `uname -r`, profile and manager flavor; unmatched
   builds fail closed.
3. **64560**: KASLR → SELinux permissive → fsuid → two-stage cred write →
   LKM load.
4. **98260**: UAF page hold → rootd → socket handoff → cred write → LKM load.
5. The manager LKO is loaded and registered; reboot clears the temporary
   root, kernel modules and working directory.

## Scope and safety

- No partition writes, no persistent root, no bootloader unlock.
- The app does not install an LSPosed manager APK; the hotfix is device-side.
- Kernel modules are unsigned and loaded at runtime; a failed chain can panic
  or reboot the device.
- Only use on devices you own or are explicitly authorized to test.

## License

Original code is AGPL-3.0 (`LICENSE`). Third-party components keep their own
licenses; see `NOTICE.md`, `THIRD_PARTY_NOTICES.md` and `LICENSES/`.
