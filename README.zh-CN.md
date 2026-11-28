# MagicRoot

[English](README.md) | 简体中文

面向授权设备的 Android 应用与原生源码，提供一次性、重启后失效的临时 root
工作流。应用不解除 bootloader、不修改分区、不创建持久化 root。

> 仅限授权测试。工作流会加载未签名内核模块、临时修改 SELinux 状态，并可能
> 导致 panic 或重启。请只在本人的设备或获得明确授权的设备上使用。

## 功能

- 两条隔离的链：CVE-2026-64560（分阶段流水线）与 CVE-2026-98260
  （UAF/rootd）。
- 多管理器后端：KernelSU、BakaSU、FinaSU、SukiSU Ultra、KernelSU-Next、
  SukiSU Nova、MapleSU，可在 UI 切换。
- 运行时 LKO 修补：按当前内核的符号表把未定义符号改为 `SHN_ABS`，并修正
  vermagic。
- 从已安装的管理器 APK 提取管理器 LKO。
- 6.18 自适应 profile：从本机 `boot.img` 生成只对同一 `uname -r` 生效的
  profile。
- STP 平行空间清理与设备端 LSPosed 热修复。

## 目录

```
main/compose-app/   Compose UI 与控制器
main/app/           设备脚本、资产、原生代码、build.ps1
binary-sources/     原生 payload 源码与 per-build 头文件
tools/              构建、审计、打包与测试脚本
LICENSES/           第三方许可证全文
docs/               架构、构建与发布说明
```

本源码快照**不包含**预编译的内核模块与 payload 二进制；需从
`binary-sources/` 重新构建或自行提供。

## 构建

Release 构建（Windows，JDK 25 + Android SDK Platform 37；需要完整运行资产）：

```powershell
powershell -ExecutionPolicy Bypass -File .\main\app\build.ps1 `
  -Sdk <sdk> -Jdk <jdk25> `
  -Keystore <release.keystore> `
  -StorePass <storePass> -KeyAlias magicroot -KeyPass <keyPass>
```

静态检查（需要完整运行资产）：

```sh
bash tools/check.sh
```

Gradle 工程位于 `main/compose-app/`；应用资产从 `main/app/` 暂存。详见
`docs/build.md`。

## 工作方式

1. 通过 shell/Shizuku 路径把选中的资产部署到 `/data/local/tmp/magic`。
2. 按 `uname -r`、profile 与管理器 flavor 筛选资产；未匹配的 build
   fail-closed。
3. **64560**：KASLR → SELinux permissive → fsuid → 两阶段 cred 写 → LKM
   加载。
4. **98260**：UAF 页面持有 → rootd → socket 交接 → cred 写 → LKM 加载。
5. 加载并注册管理器 LKO；重启后临时 root、内核模块与工作目录全部消失。

## 范围与安全

- 不写分区、不做持久化 root、不解除 bootloader。
- 不安装 LSPosed 管理器 APK；热修复只在设备端执行。
- 内核模块未签名并在运行时加载；链失败可能导致 panic 或重启。
- 请只在本人的设备或获得明确授权的设备上使用。

## 许可

自研代码为 AGPL-3.0（`LICENSE`）。第三方组件沿用各自协议；见
`NOTICE.md`、`THIRD_PARTY_NOTICES.md` 与 `LICENSES/`。
