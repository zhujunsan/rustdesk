# SanRDesk macOS Flutter 换名清点

本文件只做盘点，**不改产品代码**。范围：本机 macOS 的 Flutter 包（`build.py --flutter` → `flutter build macos`）。

目标品牌：

| 项 | 值 |
|---|---|
| 显示名 | `SanRDesk` |
| Bundle ID | `com.san.sanrdesk`（建议值，尚未拍板） |
| URL scheme | `sanrdesk://`（由 `APP_NAME` 转小写得到，须与 Info.plist 一致） |

硬约束：

- **不要**改 Cargo crate 名 `rustdesk`，也不要改 `[lib] name = "librustdesk"`。
- 包内 `Contents/Frameworks/liblibrustdesk.dylib` **可以**改成 `libsanrdesk.dylib`（见第 5b 节）：只动嵌入文件名和 Mach-O install name，不改 crate。
- `build.py` 里复制为 `librustdesk.dylib` 的那一步仍不动（那份副本不进 `.app`）。
- Windows / Linux / iOS / Android / Sciter / 安装包脚本：**本轮一律不碰**。

两条身份必须对齐，否则装服务、深链、Dock 名会对不上：

1. **Xcode `PRODUCT_NAME`**：决定 `.app` 包名、可执行文件名、Swift 模块名、Finder/Dock 显示名。
2. **运行时 `hbb_common::config::APP_NAME`**：决定配置目录、launchd 标签、`get_uri_prefix()`、托盘文案、翻译替换、`is_custom_client()`。

把 `PRODUCT_NAME` 改成 `SanRDesk` 而 `APP_NAME` 仍是 `RustDesk`，Daemon 会去找 `/Applications/RustDesk.app`；反过来，plist 会去找不存在的 `SanRDesk.app`。

---

## 1. `flutter/macos/Runner/Configs/AppInfo.xcconfig`

Xcode Runner 目标以本文件为 `baseConfigurationReference`。

| 键 | 当前值 | 建议 SanRDesk 值 |
|---|---|---|
| `PRODUCT_NAME` | `RustDesk` | `SanRDesk` |
| `PRODUCT_BUNDLE_IDENTIFIER` | `com.carriez.flutterHbb` | `com.san.sanrdesk`（建议） |
| `PRODUCT_COPYRIGHT` | `Copyright © 2026 Purslane Tech Pte. Ltd. All rights reserved.` | 按实际版权改；会进「关于」面板和 Finder「简介」 |

**关键陷阱：** Runner 目标在 `project.pbxproj` 里又写死了 `PRODUCT_BUNDLE_IDENTIFIER = com.carriez.rustdesk`。Xcode 目标级设置会覆盖 xcconfig，所以 **只改本文件的 Bundle ID 不会生效**。真正进包的是 pbxproj 里那三处（Debug / Release / Profile）。

`PRODUCT_NAME` 没有在 pbxproj 目标里再覆盖，改 xcconfig 即可改变输出包名。

| 若跳过 | Finder / Dock / 菜单栏仍叫 RustDesk；Swift 模块名也不变。 |
| 合官方冲突 | 中。官方偶尔改版权年份或 Bundle ID；rebase 时按键名手改，不要整文件覆盖。 |

---

## 2. `flutter/macos/Runner/Info.plist`

| 键 | 当前值 | 建议 SanRDesk 值 |
|---|---|---|
| `CFBundleName` / `CFBundleExecutable` / `CFBundleIdentifier` | `$(PRODUCT_NAME)` / `$(EXECUTABLE_NAME)` / `$(PRODUCT_BUNDLE_IDENTIFIER)` | 不用改，跟 xcconfig + pbxproj |
| `CFBundleIconFile` | `AppIcon.icns` | 文件名可保持，换文件内容（见第 8 节） |
| `CFBundleURLName` | `com.carriez.rustdesk` | `com.san.sanrdesk`（建议，与 Bundle ID 对齐） |
| `CFBundleURLSchemes` | `rustdesk` | `sanrdesk` |
| `NSHumanReadableCopyright` | `$(PRODUCT_COPYRIGHT)` | 不用改 |

`src/common.rs` 的 `get_uri_prefix()` 是 `format!("{}://", get_app_name().to_lowercase())`。`APP_NAME=SanRDesk` 时运行时 scheme 变成 `sanrdesk://`。Dart 侧 `bind.mainUriPrefixSync()` 走这条路径，**Info.plist 必须同步**，否则 `sanrdesk://` 深链不会进本应用。

| 若跳过 | 系统仍注册 `rustdesk://`，与运行时 `sanrdesk://` 脱节；已有 RustDesk 安装时还会抢 scheme。 |
| 合官方冲突 | 低。这两处很少动。 |

---

## 3. `flutter/macos/Runner/Base.lproj/MainMenu.xib`

两处 `customModule="RustDesk"`（约第 16、333 行）：

- `AppDelegate`（`customClass="AppDelegate"`）
- `MainFlutterWindow`（`customClass="MainFlutterWindow"`）

Swift 模块名默认等于 `PRODUCT_NAME`。`PRODUCT_NAME=SanRDesk` 后模块就是 `SanRDesk`，**这里必须改成 `customModule="SanRDesk"`**，否则 Nib 反序列化失败，窗口起不来。

菜单里的 `APP_NAME` / `About APP_NAME` / `Quit APP_NAME` 是 Cocoa 占位符，运行时用 `CFBundleName` 替换，**不要手改这些字符串**。

| 若跳过 | 改了 `PRODUCT_NAME` 之后 macOS 包可能直接打不开。 |
| 合官方冲突 | 中。官方若动过 xib，冲突会落在 `customModule` 这一属性上，保留 `SanRDesk`。 |

---

## 4. `flutter/macos/Runner.xcodeproj/project.pbxproj`

| 位置 | 当前值 | 建议 SanRDesk 值 |
|---|---|---|
| `PBXFileReference` `path`（约第 63 行）及 Products 组、`productReference` | `RustDesk.app` | `SanRDesk.app` |
| Runner 目标 Debug / Release / Profile 的 `PRODUCT_BUNDLE_IDENTIFIER`（约第 448、593、630 行） | `com.carriez.rustdesk` | `com.san.sanrdesk`（建议） |
| Frameworks / Embed Libraries / `PBXFileReference`（约第 29、30、51、77、89、175 行） | `liblibrustdesk.dylib` | `libsanrdesk.dylib`（见第 5b 节；须与 install name 一起改） |

Xcode 目标名仍是 `Runner`，`productName = Runner` 可保持。真正的包名由 `PRODUCT_NAME` 决定；Products 组的 `path = RustDesk.app` 主要影响 Xcode 导航和 scheme 的 `BuildableName`。

同目录 scheme 也硬编码了包名，建议一并改：

`flutter/macos/Runner.xcodeproj/xcshareddata/xcschemes/Runner.xcscheme`

四处 `BuildableName = "RustDesk.app"` → `SanRDesk.app`（Build / Test / Launch / Profile）。不改的话用 Xcode 点运行可能找不到产物；`flutter build macos` 命令行一般仍能编过。

`DEVELOPMENT_TEAM = HZF9JMC8YN` 是官方团队。本机 ad-hoc（`CODE_SIGN_IDENTITY = "-"`）可暂留；要用个人证书再换。**不是显示名问题。**

| 若跳过 `RustDesk.app` path | 命令行 Flutter 构建多半仍产出 `$(PRODUCT_NAME).app`；Xcode GUI / scheme 会乱。 |
| 若跳过 Bundle ID | 运行时仍是 `com.carriez.rustdesk`，和建议 ID、TCC 权限记录不一致。 |
| 合官方冲突 | 高。pbxproj 几乎每次官方 macOS 改动都会碰；rebase 时只重放上述几处，不要整文件合。 |

---

## 5. `build.py`（Flutter macOS 拷 `service`）

`build_flutter_dmg()` 约第 909 行：

```text
cp .../target/release/service ./build/macos/Build/Products/Release/RustDesk.app/Contents/MacOS/
```

应改为 `SanRDesk.app`。这是 Flutter macOS 装后台服务的必要一步：`daemon.plist` 的 ProgramArguments 指向 `.../Contents/MacOS/service`。

同函数里被注释掉的 `create-dmg` 仍写 `RustDesk.app` / `RustDesk Installer`。本机若要用 DMG，一起改；只编 `.app` 可先不动。

同仓库其它硬编码（**本轮不碰**，仅备案）：

| 文件 | 用途 |
|---|---|
| `res/osx-dist.sh` | 签名 / `create-dmg`，路径仍是 `RustDesk.app` |
| `.github/workflows/flutter-build.yml` | CI 的 `create-dmg` / `sign-macos-app.sh` 同样写死 `RustDesk.app` |
| `build.py` 约 1072 行以后 | Sciter `cargo bundle` 的 macOS 路径，不是 Flutter |

dylib 这一行不要改（见第 5b 节；这份副本不进 `.app`）：

```text
cp target/release/liblibrustdesk.dylib target/release/librustdesk.dylib
```

| 若跳过第 909 行 | 编出的 `.app` 没有 `service`，无法装 LaunchDaemon。 |
| 合官方冲突 | 中。`build_flutter_dmg` 不常改，但一改就会撞这一行。 |

---

## 5b. 包内 dylib 改名（`liblibrustdesk.dylib` → `libsanrdesk.dylib`）

Cargo `[lib] name = "librustdesk"` **不要改**。改它会连带 Linux/Windows/Android 的 `librustdesk.so` / `.dll`，以及 `use librustdesk::*`。macOS Flutter 也不按文件名 `dlopen`：Dart 走 `DynamicLibrary.process()`，Swift 调的是 `rustdesk_core_main()`，靠 Xcode 把 dylib 链进 Runner。

dyld 找的是 Mach-O **install name**（一般是 `@rpath/liblibrustdesk.dylib`），再去 `Contents/Frameworks/` 对文件名。只 `mv` 已经编好的 `.app` 里的文件、或只改磁盘名不改 `-id` / 不让 Xcode 重新链，启动会 `Library not loaded`。

做法：cargo 仍产出 `liblibrustdesk.dylib`，编完再拷一份、改 id，让 Xcode 嵌那份新名字。三条：

1. `cargo build` 之后、`flutter build macos` 之前（写进 `build.py` 的 `build_flutter_dmg`，紧挨现有那行 `cp` 之后）：

```bash
cp target/release/liblibrustdesk.dylib target/release/libsanrdesk.dylib
install_name_tool -id @rpath/libsanrdesk.dylib target/release/libsanrdesk.dylib
```

2. `flutter/macos/Runner.xcodeproj/project.pbxproj` 里所有 `liblibrustdesk.dylib` 改成 `libsanrdesk.dylib`（Frameworks 引用、Embed Libraries、`path = ../../target/release/...`）。约第 29、30、51、77、89、175 行。

3. **不要**改 `build.py` 现有的 `cp ... librustdesk.dylib`。那份副本 Flutter macOS 不用。

不要只做的事：只把 `.app` 里的文件改名；或只 `install_name_tool -id` 而不改 pbxproj（Runner 的 `LC_LOAD_DYLIB` 仍是旧名）。

编完确认：

```bash
otool -L SanRDesk.app/Contents/MacOS/SanRDesk | grep dylib
ls SanRDesk.app/Contents/Frameworks
```

可执行文件应出现 `@rpath/libsanrdesk.dylib`，Frameworks 里也是同名文件。

这只藏 Finder / `ls` 的文件名。二进制里仍有 `rustdesk_core_main`、符号修饰里的 `librustdesk`、字符串和协议指纹。

另一种（本轮不做）：改成静态库 `.a`（iOS 已如此），`Frameworks/` 下不再有这份 dylib。比改名更重，要重做链接方式和弱链。

| 若跳过 | `.app` 里仍是 `liblibrustdesk.dylib`。 |
| 合官方冲突 | 高。pbxproj 每次官方 macOS 改动都可能碰；rebase 时只重放文件名那几处。`build.py` 新增的两行按「跟在现有 `cp` 后面」重放。 |

---

## 6. `libs/hbb_common` 的 `APP_NAME`，以及 `get_app_name()` / `is_custom_client()`

`libs/hbb_common` 是 submodule（`.gitmodules` → `https://github.com/rustdesk/hbb_common`）。本工作区当前**已经 checkout**，不是空目录。

打算改的文件：`libs/hbb_common/src/config.rs`

| 符号 | 当前值 | 建议 SanRDesk 值 |
|---|---|---|
| `APP_NAME`（约第 72 行） | `"RustDesk"` | `"SanRDesk"` |
| `ORG`（约第 57 行，仅 macOS） | `"com.carriez"` | **本轮建议保持**（见下） |

`ORG` 不要跟着 Bundle ID 改成 `com.san`，除非同时改 `src/platform/macos.rs` 里写死的 `/Library/LaunchDaemons/com.carriez.{}_service.plist` 等路径。`get_full_name()` 是 `{ORG}.{APP_NAME}`，现在会变成 `com.carriez.SanRDesk`，和 `write_plists()` 的 `com.carriez.{APP_NAME}` 一致。只改 `ORG` 会让 launchd 路径对不上。

macOS 上 `APP_NAME` 还会走进：

- 配置：`~/Library/Preferences/com.carriez.SanRDesk/`（`ProjectDirs` 的 Application Support 被 `patch()` 换成 Preferences）
- 日志：`~/Library/Logs/SanRDesk/`
- IPC：`/tmp/SanRDesk-{uid}`、`/tmp/SanRDesk-service`

父仓库行为：

```1062:1074:src/common.rs
pub fn get_app_name() -> String {
    hbb_common::config::APP_NAME.read().unwrap().clone()
}

pub fn is_rustdesk() -> bool {
    hbb_common::config::APP_NAME.read().unwrap().eq("RustDesk")
}

pub fn get_uri_prefix() -> String {
    format!("{}://", get_app_name().to_lowercase())
}
```

```2459:2462:src/common.rs
pub fn is_custom_client() -> bool {
    get_app_name() != "RustDesk"
}
```

`APP_NAME` 一旦不是 `"RustDesk"`：

- `is_custom_client()` / Dart `bind.isCustomClient()` 为 true。
- `src/lang.rs` 开始把翻译里的 `RustDesk` 换成 `SanRDesk`（见第 11 节例外）。
- **跳过官方软件更新检查**（`check_software_update()` 直接 return）。
- 主页下载/更新卡片、部分「关于/升级」入口被 `!isCustomClient()` 藏掉。
- 窗口标题、托盘 tooltip 走 `get_app_name()`，会变成 SanRDesk。

`src/common.rs` 里还有运行时覆盖：签名过的 custom client JSON 的 `app-name` 会写入 `APP_NAME`。本分叉若把默认值改掉，就不依赖那条路径。

**不要改** `src/common.rs` 的比较字符串 `"RustDesk"`——那是「是不是官方客户端」的探针，改了会让 `is_custom_client()` 永远为 false。

| 若跳过 | Dock 可能已叫 SanRDesk，配置目录、深链、托盘、翻译仍是 RustDesk。 |
| 合官方冲突 | 高。这是 submodule。官方 bump 时先看 commit range，再把 `APP_NAME` 默认值重放上去；不要把整个 submodule 指针随便快进。 |

---

## 7. `src/platform/macos.rs` 的 `correct_app_name()` 与 Daemon/Agent plist

**结论：daemon/agent plist 以及三份 AppleScript 的源文件不必为换名而改。** 它们在嵌入后、写入磁盘前都会经过 `correct_app_name()`。

```305:313:src/platform/macos.rs
fn correct_app_name(s: &str) -> String {
    let mut s = s.to_owned();
    if let Some(bundleid) = get_bundle_id() {
        s = s.replace("com.carriez.rustdesk", &bundleid);
    }
    s = s.replace("rustdesk", &crate::get_app_name().to_lowercase());
    s = s.replace("RustDesk", &crate::get_app_name());
    s
}
```

调用点：`install.scpt`、`update.scpt`、`uninstall.scpt`、`daemon.plist`、`agent.plist`（安装、更新、卸载、`write_plists()`）。

源文件里仍是官方字符串，运行时在 `APP_NAME=SanRDesk` 且 Bundle ID=`com.san.sanrdesk` 时大致变成：

| 源（节选） | 运行时替换后 |
|---|---|
| `/Applications/RustDesk.app/.../RustDesk` | `/Applications/SanRDesk.app/.../SanRDesk` |
| `/Applications/RustDesk.app/.../service` | `/Applications/SanRDesk.app/.../service` |
| `com.carriez.RustDesk_service` / `_server` | `com.carriez.SanRDesk_service` / `_server` |
| `AssociatedBundleIdentifiers` = `com.carriez.rustdesk` | `com.san.sanrdesk`（先按完整 bundle 替换） |
| `/var/log/rustdesk_service.{err,out}` | `/var/log/sanrdesk_service.{err,out}` |
| osascript 提示 `RustDesk wants to install daemon...` | `SanRDesk wants to install daemon...` |
| `pgrep -x 'RustDesk'`（`update.scpt`） | `pgrep -x 'SanRDesk'`（须等于可执行文件名 = `PRODUCT_NAME`） |

`macos.rs` 自己拼的安装路径也用 `APP_NAME`：

- `/Library/LaunchDaemons/com.carriez.{APP_NAME}_service.plist`
- `/Library/LaunchAgents/com.carriez.{APP_NAME}_server.plist`
- 静默更新脚本里 `pgrep -x {app_name}`

因此：**plist 源文件不用改**；真正要保证的是 `APP_NAME`、`PRODUCT_NAME`、Info.plist 可执行文件名三者相同。

仍会留下的非 UI 字符串（本轮可接受）：

- launchd 标签前缀仍是 `com.carriez.SanRDesk_*`（不是 `com.san`）
- 更新临时目录 `/tmp/.rustdeskupdate-*`、`rustdesk_update.sh`、日志里的 `RustDesk GUI process did not stop...`

`src/platform/privileges_scripts/{daemon,agent}.plist`、`{install,update,uninstall}.scpt` 若被官方改动，rebase 冲突按「保留官方结构 + 继续走 `correct_app_name`」处理，不要在源里写死 SanRDesk。

| 若跳过本节（且已改 APP_NAME） | 无额外风险，替换是自动的。 |
| 若只改 plist 源、不改 APP_NAME | 运行时仍会被替换回 `get_app_name()`，白改。 |
| 合官方冲突 | plist/scpt 中低；`macos.rs` 高，但换名不必改它。 |

---

## 8. 图标：`AppIcon.icns`

| 路径 | 现状 | 建议 |
|---|---|---|
| `flutter/macos/Runner/AppIcon.icns` | pbxproj / Info.plist 引用；**当前 checkout 里没有这个文件** | 用 SanRDesk 图标生成同名 `.icns` 放到该路径 |
| `flutter/macos/Runner/Info.plist` → `CFBundleIconFile` | `AppIcon.icns` | 保持文件名，只换内容 |
| `flutter/assets/icon.png`、`logo.png`、`logo_light.png`、`logo_dark.png` | 根 `.gitignore` 有 `*png`，git 不跟踪；托盘和窗口内图标走这里 | 本机放入 SanRDesk 资源；`loadIcon()` / `src/tray.rs` 读 `flutter_assets/assets/icon.png` |
| `res/32x32.png` 等 | `Cargo.toml` `[package.metadata.bundle]` 给 **Sciter** `cargo bundle` 用 | 本轮不碰 |

生成：可用 `iconutil` 或 ImageMagick，从 1024 源图出 `.icns`。根目录 `res/gen_icon.sh` 只出 png/ico，不管 `.icns`。

| 若跳过 | Dock / Finder / 关于面板仍是官方图标（若构建机能弄到官方 icns）；或 Xcode 缺资源告警。 |
| 合官方冲突 | 低（二进制）。官方更新图标时用我们的 icns 盖回去。 |

---

## 9. Flutter 设置「关于」页与 rustdesk.com 链接

桌面关于页：`flutter/lib/desktop/pages/desktop_setting_page.dart`（约 2535–2583 行）

| 当前 | 建议 | 备注 |
|---|---|---|
| `translate('About RustDesk')` | 不用改源 | `APP_NAME` 一变，`lang.rs` 会显示「About SanRDesk」 |
| `https://rustdesk.com/privacy.html` | 换成 SanRDesk 隐私政策 URL，或先拿掉可点链接 | **不会**被 `lang.rs` 替换 |
| `https://rustdesk.com`（Website） | 换成自有站点，或隐藏 | 同上 |
| `Copyright © … Purslane Tech Pte. Ltd.` | 换成实际版权 | 硬编码，可见 |

同文件附近 macOS 仍看得到的链接：

| 文件 | 当前 | 建议 |
|---|---|---|
| `flutter/lib/common.dart` `loadPowered()` 约 3741 | 点击打开 `https://rustdesk.com`；文案 `translate("powered_by_me")` | 见第 11 节：`powered_by_me` **故意不替换** |
| `flutter/lib/desktop/pages/connection_page.dart` 约 44 | `https://rustdesk.com/pricing` | 自定义客户端仍可能点到；改 URL 或藏入口 |
| `flutter/lib/desktop/pages/install_page.dart` 约 190 | `https://rustdesk.com/privacy.html` | 安装流程可见 |
| `flutter/lib/desktop/pages/desktop_home_page.dart` 约 438 | `https://rustdesk.com/download` | 已被 `!isCustomClient()` 挡住，`APP_NAME` 一改会消失 |

移动端 `flutter/lib/mobile/pages/settings_page.dart` 的 `rustdesk.com`：**本轮不碰**。

| 若跳过 | 关于页标题会变成 SanRDesk，但隐私/官网仍进 rustdesk.com。 |
| 合官方冲突 | 中。关于页布局偶尔改；rebase 时只重放 URL 和版权行。 |

---

## 10. 2FA issuer：`src/auth_2fa.rs`

```17:18:src/auth_2fa.rs
const ISSUER: &str = "RustDesk";
const TAG_LOGIN: &str = "Connection";
```

写入 TOTP 的 issuer 是 `"RustDesk Connection"`，会出现在 Google Authenticator / 系统密码里。`lang.rs` **不管**这段。

建议：`ISSUER` 改为 `"SanRDesk"`。已扫过的旧账号不会自动改名。

| 若跳过 | Authenticator 里仍显示 RustDesk。 |
| 合官方冲突 | 低。这一行几乎不动。 |

---

## 11. `APP_NAME` 替换之后，macOS 上仍会露出的 “RustDesk”

`src/lang.rs` 在 `!is_rustdesk()` 时把翻译值里的 `RustDesk` 换成 `get_app_name()`，因此 **不必改 `src/lang/*.rs`**。两个例外：

1. key `powered_by_me` — **不替换**，英文仍是 `Powered by RustDesk`（`src/lang/en.rs`）。
2. key 以 `upgrade_rustdesk_server_pro` 开头的 — **不替换**。

`hide-powered-by-me=Y` 可藏掉角标；不藏的话 macOS 主界面仍有 “Powered by RustDesk” 并链到 rustdesk.com。要换名就把 `en.rs` 这条改掉，或继续靠内置选项藏。

其它 macOS 可见 / 半可见残留：

| 文件 | 当前 | 要不要改 | 若跳过 |
|---|---|---|---|
| `flutter/lib/common.dart` `loadPowered` | 文案 + rustdesk.com | 建议改或藏 | 角标仍写 RustDesk |
| `src/lang/en.rs` `powered_by_me` | `Powered by RustDesk` | 若要显示 SanRDesk 才改 | 同上 |
| `src/auth_2fa.rs` | 见第 10 节 | 要 | Authenticator 品牌错误 |
| 关于页 URL / 版权 | 见第 9 节 | 要 | 外链仍是官方 |
| `flutter/macos/Runner/MainFlutterWindow.swift` | `NSLog("[RustDesk] …")`、插件名 `RustDeskPlugin` | 日志可选；**插件名不要改**（通道名） | Console.app 仍出现 RustDesk |
| `flutter/lib/desktop/widgets/tabbar_widget.dart` 约 644 | 硬编码 `"RustDesk"` | **本轮可跳过**：`offstage: kUseCompatibleUiMode \|\| isMacOS`，macOS 不画 | 无 |
| `PRODUCT_COPYRIGHT` / 关于页 Purslane | 官方公司名 | 建议改 | 「简介」仍是 Purslane |
| `Contents/Frameworks/liblibrustdesk.dylib` | 包内唯一带 rustdesk 的文件名 | 见第 5b 节 | `ls` / Finder 仍能看到 rustdesk |
| launchd 标签 `com.carriez.SanRDesk_*` | `correct_app_name` 故意留下的前缀 | 本轮可接受 | `launchctl list` 里仍有 carriez |
| 系统 TCC 权限列表 | 显示 Bundle 名 + 图标 | 靠 PRODUCT_NAME + icns + Bundle ID | 权限条目可能还叫旧名（换 Bundle ID 会当成新应用） |

**不要全局替换、也不要改的（macOS 本轮）：**

- Cargo `[package] name = "rustdesk"`、`[lib] name = "librustdesk"`、`default-run = "rustdesk"`（包内 dylib **文件名**按第 5b 节改，crate 名不动）
- 类型名：`RustDeskTerminal`、`RustDeskMultiWindowManager`、`RustDeskPlugin`、`isRustDeskIdd` 等
- 默认 rendezvous：`rs-ny.rustdesk.com`、`https://admin.rustdesk.com`（服务端问题，不是显示名）
- `src/lang/*.rs` 全量翻译（运行时已替换；`it.rs` 更不能填）

---

## 本轮明确不碰的平台 / 路径

以下只备案，**不要改**：

- `flutter/ios/**`（含 `Info.plist` 里的 `RustDesk`）
- `flutter/android/**`
- `flutter/windows/**`、`flutter/linux/**`
- `res/msi/**`、`res/*.spec`、`res/DEBIAN/**`、`res/rustdesk.desktop`
- `Cargo.toml` 的 `[package.metadata.winres]`、`[package.metadata.bundle]`（Windows / Sciter）
- `build.py` 的 Windows / Linux / Sciter / deb / dmg-sciter 分支
- `libs/portable/**`

---

## 建议落地顺序（先编官方，再换名）

与 `sanrdesk/README.md` 一致：官方树先能在本机编出，再动品牌，避免把编译问题和换名搅在一起。

1. **初始化并确认 `libs/hbb_common` 可编译**（当前已有内容；空目录时先 `git submodule update --init`）。
2. **用官方字符串编一次 Flutter macOS**（`./build.py --flutter` 或等价命令），确认 `flutter/build/macos/Build/Products/Release/RustDesk.app` 能启动。结果记在 `BUILD-STATUS.md`。
3. **先改运行时名**：`libs/hbb_common/src/config.rs` 的 `APP_NAME` → `SanRDesk`。`ORG` 先不动。
4. **再改包身份（一次改完，否则中间态编不过 / 装不了服务）：**
   - `AppInfo.xcconfig`：`PRODUCT_NAME`、建议同时改 `PRODUCT_BUNDLE_IDENTIFIER`
   - `project.pbxproj`：三处真正的 Bundle ID + `RustDesk.app` 产物路径 + 第 5b 节的 `libsanrdesk.dylib` 引用
   - `Runner.xcscheme`：`BuildableName`
   - `Info.plist`：URL scheme / URL name
   - `MainMenu.xib`：`customModule`
5. **`build.py` 第 909 行** 的 `RustDesk.app` → `SanRDesk.app`（否则 `service` 拷错地方）。同函数里按第 5b 节在现有 dylib `cp` 之后加上拷贝 `libsanrdesk.dylib` 和 `install_name_tool -id`。
6. **放入 `AppIcon.icns` 和 Flutter `assets` 图标。**
7. **扫可见残留：** `auth_2fa.rs` issuer、关于页 URL/版权、`powered_by_me`（或 `hide-powered-by-me`）。
8. **再编一次**，确认产物是 `SanRDesk.app`，Dock 名、关于页、深链 `sanrdesk://`、安装 Daemon 后的 plist 路径都对；`Contents/Frameworks/libsanrdesk.dylib` 存在，且 `otool -L` 里是 `@rpath/libsanrdesk.dylib`。
9. 每改一处在 `CHANGES.md` 记一条，方便以后 rebase。

不要在第 2 步成功前改 `PRODUCT_NAME`：Swift 模块和 xib `customModule` 绑在一起，失败时很难判断是官方编译问题还是换名问题。
