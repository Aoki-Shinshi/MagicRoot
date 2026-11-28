# MagicRoot

Android application and native sources for a temporary, reboot-cleared root
workflow on authorized devices.

> Authorized testing only. The workflow loads unsigned kernel modules,
> temporarily changes SELinux state, and may panic or reboot the device. Use
> only on devices you own or are explicitly authorized to test.

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
from `main/app/`.

## Layout

- `main/compose-app/` — UI and controller.
- `main/app/` — device scripts, assets and build script.
- `binary-sources/` — native payload sources and per-build headers.
- `tools/` — build, audit, packaging and test scripts.
- `LICENSES/` — third-party license texts.

Prebuilt kernel modules and payload binaries are not part of this source
snapshot; rebuild them from `binary-sources/` or supply your own.

## License

Original code is AGPL-3.0 (`LICENSE`). Third-party components keep their own
licenses; see `NOTICE.md`, `THIRD_PARTY_NOTICES.md` and `LICENSES/`.
