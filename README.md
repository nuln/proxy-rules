# proxy-rules

Self-hosted dual-client rule generation repository. It pulls upstream data directly and
publishes rule sets for **mihomo (Clash Meta)** and **Shadowrocket**. GitHub Actions
rebuilds the generated branches daily.

## Branch layout

| Branch | Role | Contents |
|---|---|---|
| `master` | Source and configuration | Workflows and client configuration examples |
| `individual` | Custom rule source | Flat `*.list` files shared by both clients; read-only to Actions |
| `mihomo` | Generated | Built-in sets plus `individual/*.list` in Clash classical payload format |
| `shadowrocket` | Generated | Built-in sets plus `individual/*.list` in Surge/Shadowrocket format |

Only `master` and `individual` are edited by hand. `mihomo` and `shadowrocket` are
force-pushed by CI.

## Data sources

| Generated file | Upstream source |
|---|---|
| direct/proxy/gfw/greatfire/reject.txt | [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) |
| apple/google.txt | [felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list) |
| icloud/private/tld-not-cn.txt | [Loyalsoldier/domain-list-custom](https://github.com/Loyalsoldier/domain-list-custom) |
| cncidr/lancidr/telegramcidr.txt | [Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip) |
| china_ip_list.txt | [17mon/china_ip_list](https://github.com/17mon/china_ip_list) |
| security.txt (malware/mining/phishing, deduped against all core sets) | [URLhaus](https://urlhaus.abuse.ch/) / [adblock-nocoin](https://github.com/hoshsadiq/adblock-nocoin-list) / [Phishing Army](https://phishing.army/) / [OpenPhish](https://openphish.com/) |
| tailscale.txt (Tailscale MagicDNS / CGNAT) | `individual` branch: `tailscale.list` |
| cloudflare.txt (Cloudflare Tunnel / Zero Trust / WARP / DoH) | `individual` branch: `cloudflare.list` |
| ai-coding-direct.txt / ai-coding-proxy.txt (coding AI / agents / AI app builders) | `individual` branch: `ai-coding-direct.list` / `ai-coding-proxy.list` (verified domains) |
| pt-direct.txt / pt-proxy.txt (PT trackers direct, PT web / m-team.co images proxy) | `individual` branch: `pt-direct.list` / `pt-proxy.list` (verified domains) |

> Build logic is equivalent to clash-rules' run.yml (`full:` = exact match,
> `domain:`/bare domain = suffix including itself), built directly from the sources above.

## Client configurations (on this branch)

| File | Client | Purpose |
|---|---|---|
| `mihomo.yaml` | mihomo (Clash Meta) | Standalone full example: example Reality nodes, DNS, `Select`(select)/`Auto`(url-test)/`Fallback`(fallback) groups, and 18 `rule-providers` wired to the `mihomo` branch |
| `mihomo.lite.yaml` | mihomo (Clash Meta) | Standalone lite example: same nodes/groups plus 11 essential rule providers; the `include-all` group auto-absorbs your proxies |
| `shadowrocket.conf` | Shadowrocket | Multi-node standalone config: example `[Proxy]` nodes plus `Select`(default -> Fallback)/`Auto`(url-latency-benchmark)/`Fallback`(available) groups; rules reference the `shadowrocket` branch |
| `shadowrocket.rule.module` | Shadowrocket | Rules-only module referencing the `shadowrocket` branch. Requires a policy group named `Select`; modules cannot use the built-in `Proxy` policy |

## Import URLs

```text
mihomo:                    https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/mihomo.yaml
mihomo (lite):             https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/mihomo.lite.yaml
shadowrocket (conf):       https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/shadowrocket.conf
shadowrocket (module):     https://cdn.jsdelivr.net/gh/nuln/proxy-rules@master/shadowrocket.rule.module

rule sets (mihomo):        https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/<file>
rule sets (Shadowrocket):  https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/<file>

tailscale (mihomo):        https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/tailscale.txt
tailscale (Shadowrocket):  https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/tailscale.txt
cloudflare (mihomo):        https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/cloudflare.txt
cloudflare (Shadowrocket):  https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/cloudflare.txt
ai-coding direct (mihomo):        https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/ai-coding-direct.txt
ai-coding direct (Shadowrocket):  https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/ai-coding-direct.txt
ai-coding proxy (mihomo):         https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/ai-coding-proxy.txt
ai-coding proxy (Shadowrocket):   https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/ai-coding-proxy.txt
pt-direct (mihomo):               https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/pt-direct.txt
pt-direct (Shadowrocket):         https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/pt-direct.txt
pt-proxy (mihomo):                https://cdn.jsdelivr.net/gh/nuln/proxy-rules@mihomo/pt-proxy.txt
pt-proxy (Shadowrocket):          https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/pt-proxy.txt
```

In Shadowrocket: Config -> Modules/Profiles -> `+` -> paste the URL.

## Rule source and build workflow

Custom rules are maintained on the `individual` branch, not on `master`. See the
[individual README](https://github.com/nuln/proxy-rules/blob/individual/README.md) for
the file format, the existing lists, and the steps for adding a new list.

- The build checks out `individual` and merges its `*.list` files into both generated branches.
- `individual/*.list` uses Surge-style rule lines without a policy.
- A new list also needs a `RULE-SET` entry in the client configuration that should consume it.
- Pushing to `individual` does not trigger a build. Run `mihomo.yml` / `shadowrocket.yml`
  manually from Actions, or wait for the daily 08:00 Beijing time schedule.
- Never edit `mihomo` or `shadowrocket` directly; CI force-pushes both branches.

## Cloudflare notes

`cloudflared` Tunnel, Zero Trust / WARP enrollment, and Cloudflare DoH must be routed
DIRECT. The rule source is `individual/cloudflare.list`; the master configurations
reference the generated `cloudflare.txt` immediately after `tailscale.txt` and before
`gfw`/`proxy`.

The ordering matters and is not cosmetic: `cfargotunnel.com`, `trycloudflare.com`,
`cloudflareclient.com`, `cloudflareaccess.com`, `cloudflare-gateway.com`,
`cloudflare-dns.com`, and `one.one.one.one` all already exist on the generated
`proxy.txt`/`gfw.txt` lists. Both clients stop at the first match, so `cloudflare.txt`
must win the race to keep tunnels and zero-trust mesh direct.

- mihomo: classical provider `cloudflare`, `RULE-SET,cloudflare,DIRECT`, plus
  `+.argotunnel.com`, `+.cfargotunnel.com`, `+.cloudflare-dns.com` in `fake-ip-filter`
  so `cloudflared` dials the real edge IP.
- Shadowrocket: `RULE-SET` for `cloudflare.txt` -> `Direct`.

The list carries the Tunnel endpoint range `198.41.128.0/17`, because Tunnel endpoints
(`region1/region2.v2.argotunnel.com`) resolve into it. That range has zero overlap with
`cncidr.txt`. General Cloudflare CDN Anycast ranges (e.g. `104.16.0.0/13`, `172.64.0.0/13`)
are intentionally excluded so websites behind Cloudflare stay on proxy routes.

Deliberately proxied, not in this list: `cloudflare.com`, `dash.cloudflare.com`,
`cloudflare.net`, `workers.dev`, `pages.dev`, `cloudflarestream.com`, `cloudflare.tv`,
`cloudflare-ipfs.com`, and marketing sites. These are web properties that route through
the proxy for speed and reliability.

## Tailscale notes

`*.ts.net` MagicDNS names and `100.64.0.0/10` must be routed DIRECT. The rule source is
`individual/tailscale.list`; the master configurations already reference the generated
`tailscale.txt`:

- mihomo: classical provider `tailscale` (including daemon `PROCESS-NAME` rules for Linux and macOS), `RULE-SET,tailscale,DIRECT`, MagicDNS nameserver
  policy (`+.ts.net -> 100.100.100.100`), a fake-IP filter for `+.ts.net`, and TUN `route-exclude-address` (`100.64.0.0/10`, `fd7a:115c:a1e0::/48`).
- Shadowrocket: `RULE-SET` for `tailscale.txt`, plus `*.ts.net` and `100.64.0.0/10` in
  `skip-proxy`.

iOS allows only one active VPN. Tailscale must be the active VPN when using `*.ts.net`;
Shadowrocket cannot be the active VPN at the same time. The rules still matter on macOS,
or when Shadowrocket runs as a local proxy.

### macOS: TUN and system proxy

Mihomo's `tun.route-exclude-address` (`100.64.0.0/10` and `fd7a:115c:a1e0::/48`) prevents TUN auto-routing from intercepting Tailscale mesh traffic. In pure TUN mode (system proxy disabled), all applications—including browsers and terminal utilities—reach Tailscale nodes seamlessly.

If the GUI system proxy is toggled on concurrently, HTTP requests to Tailscale addresses enter Mihomo's local proxy port (e.g. 7890) rather than the TUN adapter. Because Mihomo binds outbound direct sockets to the physical interface (`en0`), these connections cannot reach the Tailscale `utun` interface. To use both concurrently, add Tailscale domains and CIDRs to the proxy bypass list:

```bash
networksetup -setproxybypassdomains "Wi-Fi" \
  127.0.0.1/8 192.168.0.0/16 10.0.0.0/8 172.16.0.0/12 \
  localhost '*.local' '*.crashlytics.com' '<local>' '*.ts.net' 100.64.0.0/10
```

Recommended practice: keep system proxy disabled when TUN mode is active. TUN mode already intercepts all TCP/UDP traffic at Layer 3.

## PT (Private Tracker) notes

PT traffic is split into two rule sets to separate browsing from BitTorrent protocol traffic:

- `pt-direct`: Tracker announce endpoints (`tracker.*`, `tp.*`, `DOMAIN-KEYWORD,announce`) and BitTorrent infrastructure.
  Must be DIRECT so client announcements carry the real home IP for green-light port forwarding and to avoid account bans.
- `pt-proxy`: PT web interfaces (M-Team `m-team.cc`, `m-team.io`, etc.), dedicated image hosting (e.g. M-Team image domain `m-team.co`, `ptpimg.me`, `pixhost.to`, etc.), and `.torrent` file downloads.
  Routed through `Select` proxy so web pages, poster images, and torrent downloads load without GFW or Cloudflare blocking.

All active domains were verified against live DNS and HTTP endpoints on 2026-10-26. Dead or discontinued domains (e.g. `opencd.cc`, `hdchina.org`) were replaced with their active counterparts (`open.cd`, `hdchina.club`).

## Notes

- Replace example node parameters (`server`, `uuid`, `public-key`) with real values.
  Never commit real credentials.
- mihomo `GEOIP,CN` uses its auto-managed GeoIP database by default.
