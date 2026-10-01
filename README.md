# my-clash-rules

## Claude 优先级与出口检测

整个规则列表的第一、第二条分别精确匹配 `api.anthropic.com` 和
`claude.ai`，均指向 Claude 专属，先于 STUN、进程、通配及通用规则。
这两条即使后面的上游规则集更新或缺失，也不依赖其内容匹配核心目标。

Claude 核心规则、`AI-Dedicated-Supplement.list`、验证域名现在位于
`Antigravity.list` 的 AI 进程兜底之前。即使请求来自 `Antigravity.exe` 或
`language_server.exe`，命中这些目标时也优先走 Claude 专属；
未命中的 Antigravity / Gemini / Copilot 请求仍按原有 AI 规则处理。
Clash 分组在界面上的显示顺序和检测网站的 “AI” 标签不代表匹配优先级。
OpenAI 上游规则的原有位置保持不变，不把其中的 IP / ASN 规则整体前移。

为统一 Claude 检测页面左侧与 Claude 的出口，新增：

- `DOMAIN,2026.ip138.com`：左侧 IP 检测的首选请求。
- `DOMAIN,my.ip.cn`：左侧 IP 检测的备用请求。

两者均走 Claude 专属，其他应用访问这两个域名也会受影响。
按用户要求暂不调整 DNS：没有添加 `1.1.1.1` 整地址规则，
因为它也可能改变 DNS 流量；中间 Cloudflare 检测仍沿用原有分流，
本次不保证其与 Claude 出口一致。DNS 配置文件保持原样。
原有节点选择 / 地区测速、YouTube、游戏等规则和代理组定义保持不变。
其他网站仍可能使用不同出口，不把整个出口检测表的所有服务都归入 Claude。
同一节点的服务端也可能有多出口，以用户同步后的实测为准；
网页请求超时或浏览器拦截不会因为此分流调整就必然消失。

用户自行在 Mac / Windows 更新转换订阅，并保持 Claude 专属选择同一具体节点。
本次修改不刷新本地 Clash，也不添加本地脚本。

## Claude 专属域名补充（2026-10-01）

对照 https://ip.net.coffee/claude/site.html 和 Claude Code 官方网络文档，
仅向 `Clash/AI-Dedicated-Supplement.list` 添加 9 条规则：

- 后缀：`clau.de`、`claudemcpclient.com`、`claudemcpcontent.com`、`statsigapi.net`。
- 精确域名：`anthropic.com.cdn.cloudflare.net`、
  `servd-anthropic-website.b-cdn.net`、`anthropic-com.ghost.io`、
  `browser-intake-us5-datadoghq.com`、`http-intake.logs.us5.datadoghq.com`。

已有 Anthropic / Claude 核心域名及引用规则覆盖的认证、Sentry、Intercom、
Fathom 等不重复添加。未添加全局 NTP、遥测关键词、IP 网段或 ASN 兜底。
INI 引用顺序、代理组、DNS、节点 UDP 选项和本地脚本均不因此改变。

影响与边界：

- 原先命中国外媒体的 Claude MCP 域名及上述三个 CDN / 官网域名，
  在没有更早进程规则命中时改走 Claude 专属；其余新增目标也优先走该组。
- `statsigapi.net` 包含所有子域；两个 Datadog 精确地址也可能被其他应用使用。
  访问相同目标的其他应用会一起改道，不保证仅影响 Claude 进程。
  `events.statsigapi.net` 原本已走 Claude 专属，出口策略保持一致。
- 后续按用户要求调整了优先级：这些补充域名现在先于
  `Antigravity.exe` / `language_server.exe` 进程规则匹配，见上节。
- 新增规则位于 AdsPower 进程规则之前，因此 AdsPower 访问这些特定目标时
  也走 Claude 专属，其他目标继续按原有规则。

验证时成功读取了 INI 引用的全部 29 个规则清单；新增 9 个目标均在
不带进程条件的域名匹配测试中进入 Claude 专属。对旧清单域名、代表性子域
及五种进程场景进行了 78,775 组非新增目标的前后匹配比较，结果一致。
这是静态域名 / 进程检查，不包含实时 IP 解析、GEOIP / ASN 查询、节点性能
或 Mac / Windows 实际运行验证。远程上游规则今后仍可能变化。

由用户在各设备更新使用此 INI 的转换订阅后生效；本次仓库更新不会替用户
刷新本地配置，也不需要添加本地脚本。

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

1. 出口检测、STUN、`Antigravity-Auth.list` → **Claude 专属**。
2. Claude 核心规则、专属补充及验证域名 → **Claude 专属**，优先于进程兜底。
3. `Antigravity.list` → **AI**：其余 Antigravity 域名、生成式 API、更新服务，以及 Windows 进程兜底。
4. 原有 Google 通用规则及其他规则。

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
