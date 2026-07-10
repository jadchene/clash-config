# clash-config

一组自用的 Clash/Mihomo 规则增强配置，同时提供可供 Karing 使用的规则集。

本仓库只包含规则和规则提供器定义，不包含代理节点、订阅信息或完整的 Clash 配置。主要用于：

- 为 Clash Verge Rev 等 Mihomo 客户端补充自定义分流规则
- 在“默认直连”和“默认代理”两种策略之间快速切换
- 强制 IP/DNS/浏览器泄漏检测网站走代理
- 拦截广告、QUIC 以及常见 STUN 流量
- 为 Karing 提供对应的 JSON 规则集

## 文件说明

| 文件 | 用途 |
| --- | --- |
| `merge-rule-providers.yaml` | Clash Verge Rev 配置合并文件，定义 ACL4SSR 及本仓库的远程规则集 |
| `rule-direct.yaml` | 默认直连策略：指定服务走代理，其余流量最终 `DIRECT` |
| `rule-proxy.yaml` | 默认代理策略：国内及指定服务直连，其余流量最终走 `Proxy` |
| `rulesets/custom-direct.yaml` | 自定义直连域名规则 |
| `rulesets/custom-proxy.yaml` | 自定义代理域名规则 |
| `rulesets/ipcheck.yaml` | IP、DNS、WebRTC 和浏览器泄漏检测网站规则 |
| `rulesets/karing/ipcheck.json` | Karing 使用的 IP 检测网站规则集 |
| `rulesets/karing/reject.json` | Karing 使用的 STUN、QUIC 拦截规则集 |

## Clash Verge Rev 使用方法

### 1. 添加规则提供器

在订阅的配置增强中，将下面的文件作为 Merge（合并）配置使用：

```text
https://raw.githubusercontent.com/jadchene/clash-config/main/merge-rule-providers.yaml
```

该配置会加载 OpenAI、Gemini、Google、Telegram、GitHub、Docker、JetBrains、Steam 等 ACL4SSR 规则集，以及本仓库中的自定义规则集。远程规则默认每 24 小时更新一次。

### 2. 选择分流策略

根据使用习惯选择一个 Rules（规则）增强文件。

#### 默认直连

```text
https://raw.githubusercontent.com/jadchene/clash-config/main/rule-direct.yaml
```

适合仅让指定国外服务走代理的场景。OpenAI、Gemini、Google、JetBrains、Telegram、Twitter、GitHub、Docker、Steam 和 IP 检测网站走 `Proxy`，其他未命中流量最终直连。

#### 默认代理

```text
https://raw.githubusercontent.com/jadchene/clash-config/main/rule-proxy.yaml
```

适合大部分流量都走代理的场景。局域网、中国大陆流量及部分下载/CDN 域名直连，Google、GitHub、JetBrains、OpenAI 等服务走 `Proxy`，其他未命中流量最终也走 `Proxy`。

> 配置中的代理策略组名称固定为 `Proxy`。如果你的订阅使用了其他名称，例如“节点选择”或“代理”，需要将规则中的 `Proxy` 替换为实际策略组名称。

## Karing 使用方法

在 Karing 的规则集设置中添加远程规则集，并为它们指定对应动作。

### IP 检测网站走代理

```text
https://raw.githubusercontent.com/jadchene/clash-config/main/rulesets/karing/ipcheck.json
```

将该规则集的出站方式设置为代理。它覆盖常见的公网 IP、DNS 泄漏、WebRTC 泄漏、IPv6 和浏览器指纹检测网站。

### 拦截 STUN 与 QUIC

```text
https://raw.githubusercontent.com/jadchene/clash-config/main/rulesets/karing/reject.json
```

将该规则集的出站方式设置为拦截。当前会匹配：

- 域名中包含 `stun` 的请求
- UDP 443（QUIC）
- UDP 3478、5349（常见 STUN/TURN 端口）

## 自定义规则

需要增加例外域名时，直接修改对应文件：

```yaml
# 强制直连
payload:
  - DOMAIN-SUFFIX,example.com
  - DOMAIN,download.example.com
```

```yaml
# 强制代理
payload:
  - DOMAIN-SUFFIX,example.org
  - DOMAIN-KEYWORD,example
```

- 直连规则添加到 `rulesets/custom-direct.yaml`
- 代理规则添加到 `rulesets/custom-proxy.yaml`
- IP/泄漏检测网站添加到 `rulesets/ipcheck.yaml`，并同步更新 `rulesets/karing/ipcheck.json`

## 注意事项

- 规则按照从上到下的顺序匹配，命中后不会继续匹配后续规则。
- 配置使用了 `GEOSITE`、`GEOIP`、`RULE-SET` 和 `AND` 等 Mihomo 规则语法，需要客户端内核支持。
- 拦截 UDP 443 会禁用 QUIC/HTTP/3，浏览器和大多数应用通常会自动回退到 TCP/HTTP/2。
- `DOMAIN-KEYWORD,stun` 的覆盖范围较大，可能误伤域名中恰好包含该字符串的服务；如不需要严格限制 WebRTC 泄漏，可删除该规则。
- `rule-direct.yaml` 中包含广告拦截规则；`rule-proxy.yaml` 当前不包含同等的广告拦截规则。
- 本仓库规则带有明显的个人使用偏好，建议根据自己的网络环境、订阅策略组名称和实际需求调整。

## 规则来源

通用服务规则主要来自 [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)，自定义规则由本仓库维护。
