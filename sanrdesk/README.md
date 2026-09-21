# SanRDesk 二次开发记录

本目录是相对官方 [rustdesk/rustdesk](https://github.com/rustdesk/rustdesk) 的**改动账本**，不是运行时代码。

目的：以后 `git pull` / rebase 官方迭代时，能一眼看出我们动过什么、为什么动、怎么回放。

## 约定

- **产品名**：`SanRDesk`
- **原则**：不改 Cargo crate 名 `rustdesk`，不全局替换源码里的 `rustdesk`。只改用户可见品牌（App 名、Bundle ID、图标、关于页等）。
- **官方代码保持可编译**：换名之前先确认当前 `master` 能在本机编出 macOS Flutter 包。
- **每一笔产品改动**都要在 [`CHANGES.md`](CHANGES.md) 记一条：文件、改了什么、为什么、以后合官方时怎么处理。
- **不要**把构建产物、密钥、`custom.txt` 签名材料放进这里。

## 文件

| 文件 | 用途 |
|---|---|
| [`CHANGES.md`](CHANGES.md) | 已落地的二次开发改动清单 |
| [`BRANDING-PLAN.md`](BRANDING-PLAN.md) | SanRDesk 换名要动的文件（未改代码前的清点） |
| [`BUILD-STATUS.md`](BUILD-STATUS.md) | 本机官方代码编译探测结果 |
| [`UPSTREAM.md`](UPSTREAM.md) | 以后怎么合官方更新 |
| [`SERVER.md`](SERVER.md) | 自建 hbbs/hbbr 能否沿用 |

## 当前阶段

1. 记录目录已建。
2. 换名清点已写入 [`BRANDING-PLAN.md`](BRANDING-PLAN.md)。
3. 官方 macOS Flutter 包已编出（Xcode 27 需把最低系统版本提到 12.3，见 [`BUILD-STATUS.md`](BUILD-STATUS.md)）。
4. macOS Flutter 换名已落地：显示名 / `APP_NAME` / 产物为 `SanRDesk`，Bundle ID `com.san.sanrdesk`，scheme `sanrdesk://`。crate 名 `rustdesk` 未改。细节见 [`CHANGES.md`](CHANGES.md)。
