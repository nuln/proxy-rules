# proxy-rules

Self-hosted dual-client rule generation repository: builds rules directly from the original upstream data sources and outputs two rule formats for **Shadowrocket** and **mihomo**, updated daily by GitHub Actions.

## Branch layout

| Branch | Content |
|---|---|
| `master` | Two workflows (build logic fully inlined) + client configuration examples (`mihomo.yaml`, `mihomo.lite.yaml`, `shadowrocket.conf`, `shadowrocket.rule.module`) |
| `mihomo` | 15 rule files in clash payload format (flat root), force-pushed daily |
| `shadowrocket` | 15 rule files in Surge/SR format (flat root), force-pushed daily |

## Data sources (pulled directly from origin)

| Generated file | Upstream source |
|---|---|
| direct/proxy/gfw/greatfire/reject.txt | [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) |
| apple/google.txt | [felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list) |
| icloud/private/tld-not-cn.txt | [Loyalsoldier/domain-list-custom](https://github.com/Loyalsoldier/domain-list-custom) |
| cncidr/lancidr/telegramcidr.txt | [Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip) |
| china_ip_list.txt | [17mon/china_ip_list](https://github.com/17mon/china_ip_list) |
| security.txt (malware/mining/phishing, deduped against all core sets) | [URLhaus](https://urlhaus.abuse.ch/) / [adblock-nocoin](https://github.com/hoshsadiq/adblock-nocoin-list) / [Phishing Army](https://phishing.army/) / [OpenPhish](https://openphish.com/) |

> Build logic is equivalent to clash-rules' run.yml (`full:` = exact match, `domain:`/bare domain = suffix including itself), built directly from the sources above.

## Client configurations (on this branch)

| File | Client | What it is |
|---|---|---|
| `mihomo.yaml` | mihomo (Clash Meta) | Standalone full example: example Reality nodes, DNS, `Default`(select)/`Auto`(url-test)/`Fallback`(fallback) groups + 15 `rule-providers` wired to the `mihomo` branch |
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
```

In Shadowrocket: Config → Modules/Profiles → `+` → paste URL.

## Notes

- Replace example node parameters (`server`, `uuid`, `public-key`) with real values; never commit real credentials.
- mihomo `GEOIP,CN` uses its auto-managed geoip database by default.
