# 官方代码本机编译探测

**结论：SUCCESS（套用官方 aarch64 CI 的 12.3 之后）**

未改产品名。`master` @ `a5d4ef97e` 在本机可以编出 macOS Flutter `.app`。

第一次按「完全不改源码」失败：Xcode 27 拒绝 `MACOSX_DEPLOYMENT_TARGET=10.14`。  
第二次按官方 Apple Silicon CI 把最低系统版本改成 **12.3** 后，`flutter build macos --release` 成功。

产物：

`flutter/build/macos/Build/Products/Release/RustDesk.app`（约 61MB）

- `CFBundleName` = `RustDesk`
- `CFBundleIdentifier` = `com.carriez.rustdesk`
- 版本 `1.5.0`

## 环境

| 项 | 值 |
|---|---|
| 机器 | macOS 27.0 / arm64 |
| Xcode | 27.0（只接受 deployment target 12.0–27.0） |
| Rust | rustup `1.81.0` |
| Flutter | 3.24.5（已打 CI 补丁） |
| vcpkg | `/Users/san/sdk/vcpkg` @ `9e593bb18ea69cc5095e012465dcd675a822ed0d` |

## 命令

```
# 第一次（失败）：Rust 通过，Xcode 因 10.14 失败
./build.py --flutter --hwcodec --unix-file-copy-paste

# 第二次（成功）：dylib 已在，只重编 Flutter
./build.py --flutter --hwcodec --unix-file-copy-paste --skip-cargo
```

源码改动见 `CHANGES.md`「最低系统版本 10.14 → 12.3」。这不是换名，是本机 Xcode 27 要编过必须做的、和官方 CI 相同的步骤。
