# individual

Custom rule source for `proxy-rules`. This branch is read-only to CI: its files are the
source of truth and are never force-pushed by the build.

The branch is flat. Each root-level `*.list` file is shared by mihomo and Shadowrocket.
The workflows on `master` check out this branch and generate:

| Source | mihomo output | Shadowrocket output |
|---|---|---|
| `foo.list` | `foo.txt` (classical `payload`) | `foo.txt` (raw rule lines) |

## Format

Use Surge-style rule lines without a policy. Only use rule types supported by both clients:

```text
DOMAIN-SUFFIX,example.com
IP-CIDR,203.0.113.0/24,no-resolve
IP-CIDR6,2001:db8::/32,no-resolve
```

For mihomo, the generated file is a classical payload, so the corresponding provider must
use `behavior: classical`. Do not use `behavior: domain` for these files.

## Existing lists

### `tailscale.list`

Keeps Tailscale MagicDNS, Serve, and CGNAT traffic DIRECT:

```text
DOMAIN-SUFFIX,ts.net
DOMAIN-SUFFIX,tailscale.com
DOMAIN-SUFFIX,tailscale.io
IP-CIDR,100.64.0.0/10,no-resolve
IP-CIDR6,fd7a:115c:a1e0::/48,no-resolve
```

The master client configurations already reference the generated `tailscale.txt`.

### `ai-coding.list`

Coding AI, coding agents, and AI app builders only. Every domain was tested on
2026-10-20. The generated rules are proxied:

- mihomo: `RULE-SET,ai-coding,Select`
- Shadowrocket: `RULE-SET,...,ai-coding.txt,Select`

Domains intentionally excluded:

| Domain | Reason |
|---|---|
| `continue.dev` | Acquired by Cursor; site only shows an acquisition notice |
| `supermaven.com` | Acquired by Cursor; product line no longer maintained |
| `tabnine.com` | Redirects to Tricentis; no standalone product site |
| `roocode.com` | Redirects to Roomote; Roo Code Cloud is no longer operating |
| `codegen.com` | Joined ClickUp; currently a content directory |
| `sweep.dev` | No DNS record; site does not exist |
| `grit.io` | Redirects to documentation; not a coding agent service |
| `phind.com` | Cloudflare 403; general AI search, not a coding agent |

`trae.cn` is intentionally not in the proxy list. It is routed DIRECT by
`GEOIP,CN,DIRECT` / `MATCH,DIRECT`.

## Adding a new list

1. Create `name.list` in the root of this branch.
2. Add Surge-style rules without a policy.
3. Wire it in the client configuration on `master`:
   - mihomo: add a provider with `behavior: classical`, then `RULE-SET,name,<policy>`.
   - Shadowrocket: add `RULE-SET,https://cdn.jsdelivr.net/gh/nuln/proxy-rules@shadowrocket/name.txt,<policy>`.
4. Rebuild through Actions, or wait for the daily schedule.

New file names must not collide with the 15 built-in outputs: `security`, `direct`, `proxy`,
`gfw`, `greatfire`, `reject`, `private`, `icloud`, `tld-not-cn`, `apple`, `google`, `cncidr`,
`lancidr`, `telegramcidr`, and `china_ip_list`.

## Build and edit rules

- A push to this branch does not trigger a build. Run `mihomo.yml` / `shadowrocket.yml`
  manually from Actions, or wait for the daily 08:00 Beijing time schedule.
- The build strips comments; keep rule lines clean.
- Never edit the `mihomo` or `shadowrocket` branches directly. Both are force-pushed by CI.
- Output file name equals the source file name: `foo.list` becomes `foo.txt`.
