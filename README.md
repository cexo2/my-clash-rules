# my-clash-rules

## Antigravity 分流

`Clash/rules.ini` 和 `Clash/rules-dns-fixed.ini` 使用相同顺序：

1. `Antigravity-Auth.list` → **Claude 专属**：Cloud Code、Business AI Code、Google OAuth 和账号登录。
2. `Antigravity.list` → **AI**：Antigravity 域名、生成式 API、更新服务，以及 Windows 进程兜底。
3. 原有 Google 通用规则及其他规则。

Cloud Code 域名上的所有请求都走 Claude 专属，包括登录后的 API 请求；
按域名分流无法只区分其中的登录请求。
`accounts.google.com` 和 `oauth2.googleapis.com` 是共享认证域名，
其他应用访问它们也会走 Claude 专属。
进程兜底使用 `Antigravity.exe` 和 `language_server.exe`；
其他软件使用同名 `language_server.exe` 时也会匹配 AI。

更新使用上述远程配置的转换订阅后，在 Clash 中分别选择两个组的节点。
此仓库不固定具体出口，也不能保证改变账号的地区资格。
