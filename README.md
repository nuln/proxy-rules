# proxy-rules

Self-hosted dual-client rule generation repository: builds rules directly from the original upstream data sources and outputs two rule formats for **Shadowrocket** and **mihomo**, updated daily by GitHub Actions.

## Branch layout

| Branch | Content |
|---|---|
| `master` | Two workflows (build logic fully inlined) + client configuration examples (`mihomo.yaml`, `mihomo.lite.yaml`, `shadowrocket.conf`, `shadowrocket.rule.module`) |
| `mihomo` | 16 rule files in clash payload format (flat root), force-pushed daily |
| `shadowrocket` | 16 rule files in Surge/SR format (flat root), force-pushed daily |

## Data sources (pulled directly from origin)

| Generated file | Upstream source |
|---|---|
| direct/proxy/gfw/greatfire/reject.txt | [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) |
| apple/google.txt | [felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list) |
| icloud/private/tld-not-cn.txt | [Loyalsoldier/domain-list-custom](https://github.com/Loyalsoldier/domain-list-custom) |
| cncidr/lancidr/telegramcidr.txt | [Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip) |
| china_ip_list.txt | [17mon/china_ip_list](https://github.com/17mon/china_ip_list) |
| security.txt (malware/mining/phishing, deduped against all core sets) | [URLhaus](https://urlhaus.abuse.ch/) / [adblock-nocoin](https://github.com/hoshsadiq/adblock-nocoin-list) / [Phishing Army](https://phishing.army/) / [OpenPhish](https://openphish.com/) |
| tailscale.txt (Tailscale MagicDNS / CGNAT，静态) | 本仓库维护（`*.ts.net`、`tailscale.com`、`100.64.0.0/10`、`fd7a:115c:a1e0::/48`） |

> Build logic is equivalent to clash-rules' run.yml (`full:` = exact match, `domain:`/bare domain = suffix including itself), built directly from the sources above.

## Client configurations (on this branch)

| File | Client | What it is |
|---|---|---|
| `mihomo.yaml` | mihomo (Clash Meta) | Standalone full example: example Reality nodes, DNS, `Default`(select)/`Auto`(url-test)/`Fallback`(fallback) groups + 16 `rule-providers` wired to the `mihomo` branch |
| `mihomo.lite.yaml` | mihomo (Clash Meta) | Standalone lite example: same nodes/groups + 11 essential subscriptions, `include-all` group auto-absorbs your proxies |
| `shadowrocket.conf` | Shadowrocket | Multi-node standalone config: example `[Proxy]` nodes + `Select`(default → Fallback)/`Auto`(url-latency-benchmark)/`Fallback`(available) groups, rules referencing the `shadowrocket` branch |
| `shadowrocket.rule.module` | Shadowrocket | Rules-only module referencing the `shadowrocket` branch. Requires a policy group named **`Default`** in your config — modules cannot use the built-in `Proxy` policy |

## Import URLs

```
mihomo:                    https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/mihomo.yaml
mihomo (lite):             https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/mihomo.lite.yaml
shadowrocket (conf):       https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/shadowrocket.conf
shadowrocket (module):     https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/shadowrocket.rule.module

rule sets (mihomo):        https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/<file>
rule sets (Shadowrocket):  https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/<file>

tailscale (mihomo):        https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/tailscale.txt
tailscale (Shadowrocket):  https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/tailscale.txt
```

In Shadowrocket: Config → Modules/Profiles → `+` → paste URL.

## Tailscale（`*.ts.net`）直连

Tailscale 的 MagicDNS 域名、Serve 的 HTTPS，以及 tailnet 的 CGNAT 地址必须 **DIRECT**：

- 公共 DNS/DoH 解析不了 `*.ts.net`，走代理会 `dns resolve failed`；
- 代理节点也路由不到 `100.64.0.0/10`，DIRECT 拨号会 `i/o timeout`。

两个分支都会生成 `tailscale.txt`：`mihomo` 分支是 Clash **classical** payload，
`shadowrocket` 分支是 Surge/SR 规则行。

mihomo 的 `mihomo.yaml` / `mihomo.lite.yaml` 已加好：

```yaml
rule-providers:
  tailscale:
    type: http
    behavior: classical
    format: yaml
    url: https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/tailscale.txt
    interval: 86400

rules:
  - RULE-SET,tailscale,DIRECT
```

`mihomo.yaml` 还加了 MagicDNS 策略（`mihomo.lite.yaml` 是片段，没有 `dns` 段，需要时自行合并）：

```yaml
dns:
  nameserver-policy:
    "+.ts.net":
      - 100.100.100.100   # Tailscale MagicDNS
  fake-ip-filter:
    - "+.ts.net"
```

Shadowrocket 的 `shadowrocket.conf` / `shadowrocket.rule.module` 已加：

```
RULE-SET,https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/tailscale.txt,Direct
```

并在 `[General]` 的 `skip-proxy` 里加了 `*.ts.net, 100.64.0.0/10`。

> iOS 同时只能有一个 VPN 生效：要用 `*.ts.net`，Tailscale 必须是当前 VPN；
> Shadowrocket 无法和 Tailscale 同时作为 VPN 运行。规则的价值在于 macOS
> （或 Shadowrocket 作为本地代理时）不让 tailnet 流量进代理。

### macOS：TUN + 系统代理

mihomo 开 TUN 时会把 DIRECT 出口绑定到物理网卡（`en0`），而 `100.64.0.0/10`
只能走 Tailscale 的 `utun`；系统代理同时开着时，浏览器经 mihomo 访问
`https://*.ts.net` 就会超时。除了上面的规则，还要让系统代理绕过：

```bash
# 对当前网络服务（示例为 Wi-Fi）追加 bypass；其他服务同理
networksetup -setproxybypassdomains "Wi-Fi" \
  127.0.0.1/8 192.168.0.0/16 10.0.0.0/8 172.16.0.0/12 \
  localhost '*.local' '*.crashlytics.com' '<local>' '*.ts.net' 100.64.0.0/10
```

或者开 TUN 时直接关掉系统代理（TUN 已经接管全部流量）。

## Notes

- Replace example node parameters (`server`, `uuid`, `public-key`) with real values; never commit real credentials.
- mihomo `GEOIP,CN` uses its auto-managed geoip database by default.
