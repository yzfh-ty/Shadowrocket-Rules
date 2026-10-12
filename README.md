# 小火箭 Shadowrocket Config

一份开箱即用的 小火箭 Shadowrocket 规则配置
- 支持前沿的苹果智能（Apple Intelligence）
- 在新规下的香港券商访问
- 导入后添加自己的节点或订阅即可使用

## 快速开始

1. 复制配置文件的 Raw 链接：
   `https://raw.githubusercontent.com/yzfh-ty/Shadowrocket-Rules/refs/heads/main/Shadowrocket.conf`
2. 打开 Shadowrocket → 配置 → 右上角 `+` → 粘贴链接 → 下载
3. 点击已下载的配置，设为使用中（✔️）
4. 首页添加你自己的节点或订阅
5. 连通性测试，选择可用节点连接

## 分流规则

以下是本地配置及本地规则集的默认分流设计；已有的 App 策略选择可能覆盖默认值。规则按 `[Rule]` 中的顺序优先匹配。

| 分组 | 覆盖范围 | 默认策略 |
|------|----------|----------|
| 🚀 节点选择 | AI、推送、券商等分组共用的选择入口 | PROXY |
| 🧱 DNS 防泄露 | BlockHttpDNS 中的已知 HTTPDNS 端点，微信例外提前放行 | REJECT |
| 🤖 AI 服务 | ChatGPT、Claude、Google AI、Copilot、Grok、OpenRouter、Perplexity，以及列出的苹果智能 / Siri / 搜索相关端点 | 🚀 节点选择 |
| 🍏 苹果服务 | 苹果常规服务、iCloud 照片 / 邮箱及列出的 CDN 域名 | DIRECT |
| 🍎 苹果推送 | `push.apple.com` 及子域名，以及苹果 `17.0.0.0/8` 和官方 APNs IPv6 地址段的 TCP 5223 连接 | 🚀 节点选择 |
| 🏦 汇丰香港 | 汇丰香港、Reward+ 相关端点 | DIRECT |
| 🏦 香港银行 | 规则集列出的其他香港银行端点 | DIRECT |
| 📈 券商服务 | 富途 / moomoo、长桥、老虎 / TradeUP、雪盈、盈透、Schwab | 🚀 节点选择 |
| 🏠 私有网络 | 局域网及本地域名；部分网段另由 TUN 排除设置直接绕过 | DIRECT |
| 🔒 国内服务 | 微信指定端点、豆包、DeepSeek、哔哩哔哩、国内规则集、中国 IP | DIRECT |
| 🌍 非中国 | 谷歌其他服务、YouTube、Telegram、代码托管、微软、非苹果邮件及境外规则集 | PROXY |
| 🐟 漏网之鱼 | 前面规则未匹配的流量 | PROXY |

AI 规则优先于通用谷歌、微软及代码托管规则，苹果推送规则优先于通用苹果规则。`GEOIP,CN` 位于境外规则集之后，`FINAL` 最后兜底；因此“非中国”是分组名称，并不意味着所有境外 IP 都经过地理分类。

`PROXY` 使用 App 当前代理；“🚀 节点选择”是另一个可切换的分组。“🌍 非中国”和“🐟 漏网之鱼”默认直接选 `PROXY`，不会自动跟随“🚀 节点选择”的切换。

## DNS 与规则检查

主 DNS 配置为 Cloudflare DoH，备用为 Google DoH，均带 `#proxy`。直连流量开启系统 DNS；`hijack-dns` 接管列出的公共 DNS 53 端口请求。HTTPDNS 分组只控制规则集列出的端点，不能作为所有 DNS 查询均走代理的保证。

2026-10-12 已将配置更新地址、八个自建规则集 URL 和导入链接统一改为 `yzfh-ty/Shadowrocket-Rules`。配置及修改后的 `.list` 文件需要一起发布到该仓库的 `main` 分支，手机才能加载这一版本；本地修改本身不会更新远程内容。

已限定苹果推送 TCP 5223 规则的目标地址，包含 IPv4 和[苹果官方列出的四个 APNs IPv6 地址段](https://support.apple.com/en-us/102266)，并修正谷歌重写正则的域名匹配边界。IPv6 规则已预置，当前 `ipv6` 和 `prefer-ipv6` 仍为 `false`；HTTPS 重写是否生效取决于 App 的 HTTPS 解密及证书设置。

导入时默认关闭 IPv6，需要时可在「配置 → 当前配置右侧 ⓘ → 通用 → 启用 IPv6」中开启，「首选 IPv6」可以保持关闭。更新远程配置可能覆盖手机上的本地修改，更新后应重新检查开关。

完整证据、其他问题及验证范围见 [规则检查记录](RULES-REVIEW.zh-CN.md)。

## 规则集来源

- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) — 主要规则集
- [iab0x00/ProxyRules](https://github.com/iab0x00/ProxyRules) — AI 服务补充规则
- `AI.list` 包含 Google AI（AI Studio、Antigravity、DeepMind、Gemini、NotebookLM、Search Labs），优先于 `Google.list` 匹配
- `Google.list` 收录其余谷歌服务，使用「🌍 非中国」分组
- YouTube、Telegram、微软服务及代码托管（GitHub、GitLab、Atlassian）统一使用「🌍 非中国」分组；Copilot 等 AI 规则优先使用 AI 分组
- `Mail.list` 收录 Gmail、Outlook、Yahoo、Yandex 的邮件协议端点，使用「🌍 非中国」分组
- `Apple.list` 基于 blackmatrix7 Apple 规则，并配套加载 `Apple_Domain.list`，补充 iCloud 邮箱、iCloud Photos / Apple CDN 直连域名，使用「🍏 苹果服务」分组
- `HK_Broker.list` 补充富途 / moomoo / 长桥 / 老虎 / 雪盈 / 盈透 / TradeUP / Schwab 证券域名及交易 IP 段
- `HSBC_HK.list` 与 `HK_Banks_Direct.list` 收录香港银行网站及 App 服务域名

## Star History

[![Star History Chart](https://api.star-history.com/chart?repos=yzfh-ty/Shadowrocket-Rules&type=date&legend=top-left)](https://www.star-history.com/?repos=yzfh-ty%2FShadowrocket-Rules&type=date&legend=top-left)

## License

MIT
