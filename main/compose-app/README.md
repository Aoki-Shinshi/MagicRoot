# compose-app — MagicRoot UI/控制器

## 构建

```sh
./gradlew assembleDebug
```

SDK 路径通过 `local.properties` 或 `ANDROID_HOME` 指定。正式 release 构建
使用 `main/app/build.ps1`，它会先暂存 `main/app/` 的资产再调用本工程。

## 说明

- 应用资产由 `main/app/` 提供；本工程不单独包含 LKM/payload。
- 签名配置只在构建机本地保存，不进入仓库。
- 完整环境与常见问题见 `../../docs/build.md`。
