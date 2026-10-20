# individual

这个分支只放自定义规则，Actions **只读**，不会被 force-push 覆盖。

分支是**扁平**的，不放目录；根目录下每个 `*.list` 都是 mihomo 和 Shadowrocket
共用的规则文件。`master` 上的两个构建 workflow 会 checkout 本分支，并同时生成：

| 源文件 | mihomo 输出 | Shadowrocket 输出 |
|---|---|---|
| `foo.list` | `foo.txt`（classical `payload`） | `foo.txt`（原样规则行） |

## 格式

Surge 规则行，**不写策略**；只使用两端都支持的规则类型：

```text
DOMAIN-SUFFIX,example.com
IP-CIDR,203.0.113.0/24,no-resolve
IP-CIDR6,2001:db8::/32,no-resolve
```

`tailscale.list` 已经生成两端的 `tailscale.txt`，客户端配置里已经引用：

```text
DOMAIN-SUFFIX,ts.net
DOMAIN-SUFFIX,tailscale.com
DOMAIN-SUFFIX,tailscale.io
IP-CIDR,100.64.0.0/10,no-resolve
IP-CIDR6,fd7a:115c:a1e0::/48,no-resolve
```

只想加一条 **DIRECT** 规则时，直接追加到这个文件即可，不用改客户端配置，也不用改
workflow。新建 `foo.list` 也可以，但需要在 `mihomo.yaml` / `shadowrocket.conf`
里加一条 `RULE-SET,foo,DIRECT`。

## 注意

- 输出文件名 = 源文件名（`foo.list` → `foo.txt`），不能和内置 15 个文件重名。
- 规则里不写策略；策略由客户端的 `RULE-SET,...,DIRECT` / `REJECT` 决定。
- 修改后：等每天 08:00 的定时构建，或在 Actions 页面手动 **Run workflow**，
  也可以本地执行 `gh workflow run mihomo.yml` / `gh workflow run shadowrocket.yml`。
- push 到本分支**不会**自动触发构建（本分支没有 workflow 文件）。
- 不要直接改 `mihomo` / `shadowrocket` 分支，它们会被 force-push 覆盖。
