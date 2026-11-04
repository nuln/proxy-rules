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

## Rule Lists

All custom individual rules are consolidated into two structured lists:

### `custom-direct.list`

High-priority DIRECT rules that must bypass the proxy:

1. **Tailscale & MagicDNS**: Private tailnet and CGNAT (`100.64.0.0/10`, `ts.net`)
2. **Cloudflare Special Services**: Cloudflare Tunnel (`argotunnel.com`, `trycloudflare.com`), WARP enrollment, DoH endpoints, and tunnel edge CIDR (`198.41.128.0/17`)
3. **Domestic AI Services**: Chinese LLM services (DeepSeek, Kimi `kimi.ai`, Tongyi `tongyi.ai`, MiniMax `minimax.io`, GLM `zhipuai.cn`, Doubao, 01.ai, Kling, Stepfun, etc.) placed before `tld-not-cn` to prevent misrouting via `.ai`/`.io`
4. **PT Trackers**: BitTorrent tracker announce endpoints to preserve connectability and direct P2P transfers
5. **Private Domains**: Personal domain services (e.g. `nuln.dev`)

### `custom-proxy.list`

High-priority PROXY rules that must be routed through your proxy nodes:

1. **Coding AI & Agents**: Cursor, GitHub Copilot, Windsurf, Cline, Aider, Roo Code, Augment, Amp, Factory, OpenCode, OpenHands, Replit, StackBlitz, Bolt, v0, Lovable, Antigravity, Firebase Studio, Trae AI
2. **Frontier LLM Platforms**: OpenAI / ChatGPT, Anthropic / Claude, Google Gemini, xAI / Grok, Mistral, Cohere, Meta AI
3. **AI Gateways & Hubs**: OpenRouter, Perplexity, Poe, Groq, Cerebras, Together AI, Fireworks, DeepInfra, Replicate, Hugging Face
4. **Multimodal / Generative AI**: Midjourney, Suno, Udio, Runway, Pika, Luma, ElevenLabs, Civitai
5. **AI Tooling & Compute**: Ollama, LangChain, LlamaIndex, WandB, vLLM, RunPod, Modal, Vast.ai, Pinecone, Qdrant
6. **AI Productivity**: DeepL, Notion AI, Phind, Sider, Monica, Grammarly, Linear
7. **PT Web & Images**: PT site web frontends and poster image hosts (e.g. `m-team.cc`, `ptpimg.me`)
8. **Cryptocurrency & Web3**: Major exchanges (Bybit full suite, Binance, OKX, Coinbase, Gate.io, Bitget, Kraken, KuCoin, HTX, MEXC, HashKey), market analytics (TradingView, CoinMarketCap, CoinGecko, DeFiLlama, DexScreener), Web3 wallets (MetaMask, Phantom, Rabby), DEXs (Uniswap, Raydium, Jupiter), and explorers/RPCs (Etherscan, Solscan, Infura, Alchemy)

## Build and edit rules

- A push to this branch does not trigger a build. Run `mihomo.yml` / `shadowrocket.yml`
  manually from Actions, or wait for the daily 08:00 Beijing time schedule.
- The build strips comments; lines starting with `#` and blank lines are ignored.
- Never edit the `mihomo` or `shadowrocket` branches directly. Both are force-pushed by CI.
- Output file name equals the source file name: `foo.list` becomes `foo.txt`.
