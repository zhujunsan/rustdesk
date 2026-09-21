# 已落地改动

格式（每条一次提交/一批改动）：

```
## YYYY-MM-DD — 简短标题
- 文件：`path`
- 改动：做了什么
- 原因：为什么必须改官方文件，而不是只放在 sanrdesk/
- 合官方：rebase 时怎么处理（保留 / 重放 / 可能冲突）
```

## 2026-09-21 — 建立二次开发账本

- 文件：`sanrdesk/`（本目录，官方树里新增）
- 改动：只加记录文件，未改产品代码
- 原因：换名和后续功能要能从官方迭代里单独辨认
- 合官方：纯新增目录，合入时应无冲突

## 2026-09-21 — macOS 换名清点（未改产品代码）

- 文件：`sanrdesk/BRANDING-PLAN.md`
- 改动：列出 SanRDesk 要动的 macOS 品牌文件；产品代码未改
- 原因：换名前先固定清单，避免漏改 `pbxproj` Bundle ID / `MainMenu.xib` / `build.py`
- 合官方：纯新增文档。落地换名时按该文件逐项改，rebase 只重放那些官方文件

## 2026-09-21 — 本机 Xcode 27：最低系统版本 10.14 → 12.3

- 文件：`build.py`、`Cargo.toml`、`flutter/macos/Podfile`、`flutter/macos/Runner.xcodeproj/project.pbxproj`
- 改动：与官方 aarch64 CI 相同，把 `MACOSX_DEPLOYMENT_TARGET` / `platform :osx` / `osx_minimum_system_version` 提到 `12.3`。`Podfile` `post_install` 额外强制所有 Pods 为 12.3（Xcode 27 拒绝 10.13/10.14）。
- 原因：Xcode 27 支持范围是 12.0–27.0；不改则 `flutter build macos` 过不了。官方 CI 在 Apple Silicon 上本来就会临时改成 12.3。
- 合官方：若上游仍是 10.14，rebase 后按同样四文件再提一次；若官方已 ≥12.0，丢弃我们的数字，保留 `post_install` 强制项直到上游也覆盖 Pods。

## 2026-09-21 — 运行时 APP_NAME → SanRDesk

- 文件：`libs/hbb_common/src/config.rs`
- 改动：`APP_NAME` 默认值 `"RustDesk"` → `"SanRDesk"`。`ORG`（`com.carriez`）不动。submodule 指针不 bump，只脏改这一行。
- 原因：配置目录、托盘、`get_uri_prefix()`、`is_custom_client()`、Daemon 路径替换都读 `APP_NAME`；不改则包名和运行时身份会对不上。
- 合官方：submodule 被官方 bump 时按本条重放这一行，不要快进指针把无关 hbb_common 提交带进来。

## 2026-09-21 — macOS Flutter 包身份对齐 SanRDesk

- 文件：`flutter/macos/Runner/Configs/AppInfo.xcconfig`、`flutter/macos/Runner.xcodeproj/project.pbxproj`、`flutter/macos/Runner.xcodeproj/xcshareddata/xcschemes/Runner.xcscheme`、`flutter/macos/Runner/Info.plist`、`flutter/macos/Runner/Base.lproj/MainMenu.xib`
- 改动：`PRODUCT_NAME` / 产物路径 / scheme `BuildableName` → `SanRDesk`；Bundle ID → `com.san.sanrdesk`；URL scheme → `sanrdesk`；`customModule` → `SanRDesk`；pbxproj 里嵌入的 dylib 引用 `liblibrustdesk.dylib` → `libsanrdesk.dylib`。`DEVELOPMENT_TEAM` 与 `AppIcon.icns` 路径不动。
- 原因：Xcode 目标级 Bundle ID 会覆盖 xcconfig；scheme / xib 模块名必须与 `PRODUCT_NAME` 同一次改完，否则窗口起不来或 Daemon 找不到 `.app`。
- 合官方：pbxproj 冲突高。rebase 时只重放上述键，不要整文件合。

## 2026-09-21 — build_flutter_dmg：SanRDesk.app + libsanrdesk.dylib

- 文件：`build.py`（仅 `build_flutter_dmg`）
- 改动：`service` 拷进 `SanRDesk.app`。现有 `librustdesk.dylib` 那份 `cp` 后面加拷贝 `libsanrdesk.dylib` 并用 `install_name_tool -id @rpath/libsanrdesk.dylib`。crate 名 / `liblibrustdesk.dylib` 产物名不动。注释掉的 `create-dmg` 和 1072 行之后的 Sciter 路径不碰。
- 原因：不改 service 路径则包内没有 `Contents/MacOS/service`；不改 install name / pbxproj 则 dyld 仍找 `liblibrustdesk.dylib`。
- 合官方：新增两行跟在现有 dylib `cp` 后面重放；`RustDesk.app` → `SanRDesk.app` 只改 Flutter macOS 那一行。

## 2026-09-21 — macOS ad-hoc 启动：关掉 library validation 并在拷 service 后重签

- 文件：`flutter/macos/Runner/Release.entitlements`、`flutter/macos/Runner/DebugProfile.entitlements`、`build.py`（仅 `build_flutter_dmg`）
- 改动：entitlements 增加 `com.apple.security.cs.disable-library-validation`。拷完 `service` 后用现有 `sign-macos-app.sh` 做 ad-hoc 重签。
- 原因：本机 `CODE_SIGN_IDENTITY=-` + `ENABLE_HARDENED_RUNTIME` 时，主程序和 FlutterMacOS.framework 没有同一 Team ID，dyld 在启动瞬间杀掉进程（双击无窗口）。拷 `service` 还会弄坏 bundle 密封，必须后签。
- 合官方：entitlements 多一行，官方 Developer ID 整包同 Team 重签时不依赖它；`build.py` 那一行跟在现有 `service` `cp` 后面。CI 若再用 `sign-macos-app.sh` 签正式身份会覆盖 ad-hoc。

## 2026-09-21 — 藏 rustdesk.com 外链 / Powered by，改 2FA 与版权

- 文件：`flutter/lib/desktop/pages/desktop_setting_page.dart`、`flutter/lib/common.dart`、`flutter/lib/desktop/pages/connection_page.dart`、`flutter/lib/desktop/pages/install_page.dart`、`src/auth_2fa.rs`
- 改动：隐私 / Website / 定价 / 安装页 EULA 外链用 `isCustomClient` 藏掉（官方 `APP_NAME=RustDesk` 路径保持原样）。`loadPowered` 在现有 `hide-powered-by-me` 判断上加 `|| isCustomClient`。版权占位改为 `Copyright © ${year} SanRDesk`。2FA `ISSUER` → `"SanRDesk"`。未改 `src/lang/*.rs` 和移动端 `settings_page.dart`。
- 原因：这些字符串不被 `lang.rs` 的 `RustDesk` 替换；硬编码 URL / issuer 会继续露出官方品牌。
- 合官方：中。关于页布局偶尔改；rebase 时只重放 `if (!bind.isCustomClient())` 包装、版权行、`ISSUER`、以及 `loadPowered` / `setupServerWidget` 的 extra 条件。
