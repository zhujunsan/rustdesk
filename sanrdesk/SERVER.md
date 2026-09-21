# 自建服务器（hbbs / hbbr）

换名成 SanRDesk **不会改协议**。已经在跑的官方 `rustdesk-server` 可以继续用。

## 两个进程

| 进程 | 作用 | 默认端口 |
|---|---|---|
| **hbbs** | ID / 注册 / 打洞（rendezvous） | `21116` |
| **hbbr** | 打洞失败后的中继（relay） | `21117`（= hbbs 端口 + 1） |

客户端先连 **hbbs**。P2P 失败才走 **hbbr**。只部署 hbbr、没有 hbbs，客户端拿不到 ID、也完不成握手。

## 客户端怎么指过去

设置 → 网络，或等价 option：

- `custom-rendezvous-server`：hbbs 地址（ID 服务器）
- `relay-server`：hbbr 地址。留空则用 hbbs 告诉你的地址；再空则默认「hbbs 主机:端口+1」
- `key`：必须和服务器 `id_ed25519.pub` 一致

两台机器（被控 + 主控）都要填同一套。

## 和换名的关系

- `APP_NAME` / Bundle ID / 图标都不参与 hbbs/hbbr 协议。
- `APP_NAME != "RustDesk"` 时 `is_custom_client()` 为 true，主要影响官方更新检查、关于页等，**不切断自建中继**。
- 不要把客户端编进官方公钥的 `custom.txt` 里指望改服务器；自建服务器用上面三个 option 即可。

## 本机注意

本机已有 `/Applications/RustDesk.app`。SanRDesk 编出来后请换 Bundle ID，避免和官方版抢权限 / LaunchDaemon。网络设置不会自动从官方 App 拷过来，要在 SanRDesk 里再填一次服务器。
