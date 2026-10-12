# Shadowrocket 规则检查记录

检查日期：2026-10-12。对象为当前工作区的 `Shadowrocket.conf` 和规则集；默认分流表见 README。

## 结论

本地分组引用完整，12 个分组、47 条顶层规则未发现未定义策略引用，`git diff --check` 通过。检查发现的三项问题均已修正：仓库地址统一使用 `yzfh-ty/Shadowrocket-Rules`，TCP 5223 限定苹果 IPv4 和 APNs IPv6 目标，谷歌重写正则限定域名及主机边界。配置及规则集需一起发布，手机需更新配置和规则集才能使用新版本；客户端实际运行尚未验证。

## 1. 规则来源：已统一使用自己的仓库

Git 远程为 `https://github.com/yzfh-ty/Shadowrocket-Rules.git`。检查时配置中的 `update-url`、八个自建规则集 URL 和 README 快速开始链接使用 `LingJingMaster/Shadowrocket-Rules`；现已统一改为自己的仓库。下面保留的是改链接前的检查证据，不代表新地址的当前内容。

改链接前直接获取了全部 22 个规则集 / 域名集 URL，均可读取。四个已修改文件与上游远程内容不同：

| 文件 | 本地有效规则数 | 改链接前的上游远程有效规则数 |
|------|----------------|--------------------------------|
| AI.list | 68 | 51 |
| Google.list | 704 | 721 |
| Apple.list | 52 | 50 |
| Mail.list | 14 | 16 |

将改链接前的本地配置分别配上本地自建规则和上游 URL 返回的规则，进行域名顺序匹配，得到：

| 示例端点 | 使用本地规则 | 使用改链接前的上游远程规则 |
|----------|--------------|------------------|
| gemini.google.com | 🤖 AI 服务 | 🌍 非中国 |
| aistudio.google.com | 🤖 AI 服务 | 🌍 非中国 |
| imap.mail.me.com | 🍏 苹果服务 | 🌍 非中国 |
| imap.mail.icloud.com.cn | 🍏 苹果服务 | 🌍 非中国 |

原因是检查时上游远程 AI 规则没有已迁移的 Google AI 条目，而上游远程邮件规则仍包含两个 iCloud 邮箱后缀，且邮件规则集早于苹果规则集。历史来源：[上游 AI.list](https://raw.githubusercontent.com/LingJingMaster/Shadowrocket-Rules/refs/heads/main/AI.list)、[上游 Mail.list](https://raw.githubusercontent.com/LingJingMaster/Shadowrocket-Rules/refs/heads/main/Mail.list)。

原 `update-url` 会获取仍带地区及独立服务分组的[上游配置](https://raw.githubusercontent.com/LingJingMaster/Shadowrocket-Rules/refs/heads/main/Shadowrocket.conf)。现已改为自己的仓库。

后续发布时需要将同一版本的配置和修改后的自建 `.list` 一起发布到 `yzfh-ty/Shadowrocket-Rules` 的 `main` 分支，再检查发布地址返回的实际内容。仅替换仓库名称，或仅上传 `.conf`，都不能证明规则已一致。

改链接后、发布前直接检查了自己的仓库：配置文件和八个自建规则集共九个 URL 均返回 HTTP 200。当时远程 `Shadowrocket.conf`、`AI.list`、`Google.list`、`Apple.list`、`Mail.list` 与本地不同，其余四个自建规则集与本地一致。这是发布前的历史检查结果；发布后的内容需重新核对。14 个第三方规则集地址保持不变。README 中原有的远程二维码图片已移除，导入入口统一使用自己的 Raw 链接。

## 2. 匹配范围过宽：已在本地修正

### TCP 5223 已限定苹果目标

原规则 `AND,((PROTOCOL,TCP),(DST-PORT,5223)),🍎 苹果推送` 会将所有走到该规则的 TCP 5223 连接归入苹果推送，目标可以是任意服务器。

现使用五条独立 AND 规则，每条同时限定 TCP、目标端口 5223 和地址段。地址段采用[苹果官方 APNs 网络要求](https://support.apple.com/en-us/102266)中列出的范围：

| 地址族 | TCP 5223 目标范围 |
|--------|------------------|
| IPv4 | `17.0.0.0/8` |
| IPv6 | `2620:149:a44::/48` |
| IPv6 | `2403:300:a42::/48` |
| IPv6 | `2403:300:a51::/48` |
| IPv6 | `2a01:b740:a42::/48` |

IPv6 CIDR 沿用仓库 `Apple.list` 和 `Google.list` 的 `IP-CIDR` 写法。推送域名规则继续处理带域名的请求，包括 443 回退。当前 `ipv6` 和 `prefer-ipv6` 保持 `false`，IPv6 规则作为后续启用时的预置规则；未改变当前地址族开关。

静态验证包括 IPv4 / IPv6 网段内外的 TCP 5223、每个网段边界、UDP 5223，以及苹果地址的非 5223 连接。推送域名在 443 上仍命中 ApplePush.list，而非匹配地址段的 TCP 5223 连接不会仅凭端口归入苹果推送。此处验证的是规则条件，未证明客户端运行行为。

### 谷歌重写正则已转义域名并限定主机边界

原 `^https?://(www.)?g.cn` 及 `^https?://(www.)?google.cn` 中的 `.` 表示任意字符，且主机后没有边界。正则匹配验证中，`https://gxcn.example/` 和 `https://g.cn.example/` 也命中了第一条。

现使用以下正则，跳转目标及 302 状态码保持原样：

```regex
^https?://(?:www\.)?g\.cn(?::[0-9]+)?(?:[/?#]|$)
^https?://(?:www\.)?google\.cn(?::[0-9]+)?(?:[/?#]|$)
```

验证正常主机、www 前缀、端口、路径、查询参数和片段均可匹配；相似域名、追加后缀、用户信息伪装等反例不匹配。实际 HTTPS 重写是否启用，仍取决于 App 的 HTTPS 解密设置和证书状态；当前文件只有 `hostname = *.google.cn`，不能据此证明解密及重写可用。

## 3. 需要确认实际客户端行为的项目

| 项目 | 现状与影响 |
|------|------------|
| 渣打香港网页 | `HK_Banks_Direct.list` 用 URL 路径 `/hk/` 区分 `www.sc.com`，而 MITM 主机只列了 `*.google.cn`。加密连接的路径匹配不能当作已验证；域名静态模型中 `www.sc.com` 没有银行域名规则命中。应在手机规则日志中检查，或评估使用明确域名策略的范围。 |
| DNS 防泄露 | `dns-direct-system = true` 保留直连流量的系统 DNS；微信 HTTPDNS 提前放行。这些是当前策略选择，不代表配置损坏，也不能称为所有 DNS 查询均通过代理。 |
| 苹果分组切换 | iCloud CDN / CloudKit 等域名还出现在 `skip-proxy`，不能只修改苹果分组就假定所有相关流量都会切换。需结合客户端流量路径和日志验证。 |
| AI 分组覆盖范围 | `www.bing.com` 会把普通 Bing 页面也归入 AI；`mask-api.icloud.com` 是 iCloud Private Relay 端点，而非专用 AI 端点。现有规则包含共享及辅助端点，这是范围选择，需要按使用需求决定。苹果端点用途见[官方网络要求](https://support.apple.com/en-us/101555)。 |
| AI 区域与完整性 | 删除地区分组后，配置不再筛选美国、日本等出口。登录、共享资源、未列出的新端点可能采用其他分组；规则命中不能证明服务区域或实际可用性。 |

## 4. 可简化但不属于错误

“节点选择”“非中国”“漏网之鱼”都默认选 `PROXY`，但保留各自切换入口。后两者目前不会跟随“节点选择”，这是配置结构而非悬空引用。

两条翻译 API 规则现在与通用谷歌规则使用相同策略，可视为显式说明。YouTube、Telegram、微软和代码托管保留专用规则集，可以让其中的特定域名或 IP 在国内规则之前优先匹配；删除独立分组不等于应该删除这些规则集。

## 验证边界

- 完成：本地策略引用检查、差异格式检查、22 个远程规则文件获取、15 组端点 / 端口样例的顺序匹配，以及重写正则反例验证。
- 域名模型评估了 DOMAIN、DOMAIN-SUFFIX、DOMAIN-KEYWORD 和 TCP 5223 示例；未模拟 IP、GEOIP、USER-AGENT、URL-REGEX、DNS、系统绕过和客户端内部匹配细节。因此样例只说明已列规则的静态关系。
- 尚无：Shadowrocket 配置导入解析、手机规则日志、DNS 泄露测试、推送稳定性及 AI / 银行 / 券商实际连接验证。
- 后续按用户要求修正了配置和文档中的仓库地址、TCP 5223 目标范围及谷歌重写正则，并验证了正反例。IPv6 默认关闭，需要时可通过当前配置的通用设置开启；上述静态检查不代表手机已加载或实机已验证。
