# 构建说明

## 1. 环境

| 组件 | 版本/说明 |
|---|---|
| JDK | 25（build.ps1 校验） |
| Android SDK | Platform 37；Build-Tools 37.0.0 |
| Gradle | wrapper（`main/compose-app/gradlew`） |
| 原生工具链 | Android NDK r28c；`aarch64-linux-android35-clang` |
| 原生 flags | `-std=gnu11 -O2 -w -pthread -static` |

## 2. Windows Release 构建

```powershell
powershell -ExecutionPolicy Bypass -File .\main\app\build.ps1 `
  -Sdk <sdk> -Jdk <jdk25> `
  -Keystore <release.keystore> `
  -StorePass <storePass> -KeyAlias magicroot -KeyPass <keyPass>
```

脚本会：

1. 校验 SDK Platform 37、JDK 与 `main/compose-app/gradlew`；
2. 写 `main/compose-app/local.properties` 指向 SDK；
3. 从 `main/app/` 暂存资产；
4. 调用 Gradle 构建 release。

需要完整运行资产；本源码快照不包含预编译 `.ko`/payload。

## 3. Gradle 直接构建

```sh
cd main/compose-app
./gradlew assembleDebug
```

SDK 路径通过 `local.properties` 或 `ANDROID_HOME` 指定。应用资产仍需按
`build.ps1` 的方式从 `main/app/` 暂存。

## 4. 原生 payload/LKM

```powershell
powershell -ExecutionPolicy Bypass -File .\build-binaries.ps1
```

源码在 `binary-sources/`；编译输入包括 per-build 常量头与符号表。构建前确认
目标内核的 `uname -r` 与 profile token。

## 5. 静态检查与测试

```sh
bash tools/check.sh
```

`check.sh` 覆盖脚本语法、资产双向对账、Java 单测与 Flavor/6.1/98260/H1C
回归。它需要完整运行资产；资产缺失时会报 `registered asset missing on disk`。

## 6. 常见问题

- **build.ps1 报 SDK/JDK 路径**：确认传入的目录包含
  `platforms/android-37.0/android.jar` 与 `bin/java.exe`。
- **资产缺失**：先补齐 `main/app/assets/` 下的 LKM、`ksud`、符号表与脚本。
- **check.sh 失败**：多数是资产登记与磁盘不一致；按输出的 missing 列表补齐。
- **签名材料**：keystore 只在本机；不要提交到仓库。
