# my-clash-rules

## WebRTC / STUN 分流

`Clash/rules-dns-fixed.ini` 将 `stun.cloudflare.com`、`stun.voipstunt.com`、
`stun.l.google.com`、`stun1.l.google.com` 至 `stun4.l.google.com`，
以及检测中出现的 `77.72.169.212/32`、`162.159.207.0/32` 优先分配给
**Claude 专属**，与现有 OpenAI / Claude 规则共用代理组。
IP 规则仅匹配这两个公网地址，不按整个 UDP 协议或通用端口分流。
这些共享 STUN 目标被其他应用使用时也会走该组。

使用方式：

1. 更新使用此 INI 的转换订阅，保持 TUN 开启、规则模式。
2. 在 **Claude 专属** 中选择已验证支持 UDP 的具体节点。
   如需 AI 服务也使用同一出口，在 **AI** 中手动选择同一节点；
   两个分组仍独立选择，不会自动同步。
3. 关闭旧检测标签页及旧 STUN 连接后复测，确认新连接链路为 Claude 专属，
   且检测不到本地公网 IPv4 / IPv6。

这不是完整的 WebRTC 防泄露保证：其他 STUN 服务器、自定义端口和对等连接
不在这些精确规则的覆盖范围内。相同节点也可能对 TCP / UDP 使用不同出口，
最终应以实际检测为准。若新连接仍显示 DIRECT，检查客户端实际生成的规则、
本地覆写和节点 UDP 支持；仅更新本仓库不会自动刷新客户端的运行配置。

排查时还应检查内核 `/proxies` 接口返回的节点及分组 `udp` 标记。
2026-10-01 实测发现所选 VLESS 节点标记为 `udp: false`，尽管 STUN 域名规则
已经加载，规则模式下请求仍直连。在当前订阅扩展脚本中为该具体节点设置
`udp: true` 并重载后，Cloudflare 和 Voipstunt 的 STUN 请求均经 Claude 专属
返回同一代理出口。该节点开关属于客户端/订阅节点配置，本 INI 的域名规则
不会自动开启它；不能将一次测试结果推广为所有节点均支持 UDP。

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
