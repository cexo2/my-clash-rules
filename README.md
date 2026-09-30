# my-clash-rules

## Antigravity 分流

`Clash/rules-dns-fixed.ini` 使用以下分流顺序：

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

## AdsPower 分流

`Clash/rules-dns-fixed.ini` 将 `AdsPower Global.exe` 和 `AdsPowerTool.exe` 的连接
交给 **节点选择**，优先于普通直连规则和中国 IP 直连规则。
前面的专属域名规则及局域网直连规则仍优先匹配；其他进程的分流及代理组不变。

在 AdsPower 中开启“使用系统代理连接代理”，填写本机 Clash 的代理地址和端口。
浏览器环境中保留原有日本代理；Clash 保持规则模式，并为节点选择组选用可用的海外节点。
此规则只处理已经进入 Clash、且能识别所属进程的连接，不会自动接管未进入 Clash 的流量。
