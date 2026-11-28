# MagicRoot 架构

## 1. 模块

| 目录 | 职责 |
|---|---|
| `main/compose-app/` | Compose + Miuix 应用：UI、控制器、任务状态 |
| `main/app/assets/` | 设备端脚本、profile、符号表、LKM/载荷资产 |
| `main/app/native/` | 应用原生代码与 JNI |
| `main/app/res/` | 应用资源 |
| `binary-sources/` | 原生 payload 源码与 per-build 常量头 |
| `tools/` | 静态门禁、审计、打包与真机辅助 |
| `LICENSES/` | 第三方许可证 |

## 2. 资产路由

1. 读取运行内核的 `uname -r`。
2. 按 profile 与 token 选择精确资产；未匹配的 build 进入
   reserved/unsupported 分支并安全退出。
3. 按管理器 flavor（KernelSU / BakaSU / FinaSU / SukiSU / Next / Nova /
   MapleSU）选择 `ksud`、管理器 LKO 模板与符号表。
4. 只在精确匹配时部署；不借用其他内核的符号表。

## 3. 两条链

### 3.1 CVE-2026-64560

KASLR 泄漏 → SELinux permissive → fsuid → cred 两阶段写 → LKM 加载。
分阶段流水线，与 98260 链隔离。

### 3.2 CVE-2026-98260

UAF 页面持有 → rootd → socket 交接 → cred 写 → LKM 加载。适用于另一组
受支持内核。

## 4. 运行时 LKO 修补

- 每个 build 一份符号表。
- 把管理器模板 LKO 的未定义符号改写为 `SHN_ABS` 绝对地址。
- 修正 vermagic，使其匹配当前内核。
- 无法匹配的模板 fail-closed，不写入错误地址。

## 5. 管理器集成

- 优先从已安装管理器 APK 提取匹配 KMI 的 LKO / `ksud`。
- 管理器容器格式（如 FinaSU）走管理器自带的 `ksud debug extract-binary`。
- 加载后完成管理器登记、feature 下发与连接验收。

## 6. 安全模型

- 临时 root：重启后失效。
- 不写分区、不做持久化、不安装开机自启。
- 不安装 LSPosed 管理器 APK；只做设备端热修复。
- 链失败即停；不把失败状态重试成持久化状态。

## 7. 构建产物

- `main/compose-app/app/build/outputs/apk/`：APK。
- `binary-sources/` 编译出的 payload/LKM：由 `build-binaries.ps1` 或
  `main/app/build.ps1` 生成。
- 签名材料只在本机；仓库不包含 keystore。
