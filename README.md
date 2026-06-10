# Jarvis Market Signals — MCP server

Live crypto-derivatives and market signals for AI agents: perp liquidations, funding-rate crowding, open-interest positioning, safe stablecoin yield, and FX — fresh streaming data a static model can't know, served over a metered MCP endpoint.

**Endpoint:** `https://jarvis.tail12fa6b.ts.net:8443/mcp` (streamable HTTP)
**Free tier:** 100 calls/day — [open an issue](../../issues/new?title=key%20request) to get a key.
**Discovery is open:** `initialize` and `tools/list` need no key, so any MCP client or registry scanner can inspect the tools before authenticating.

## Tools

| Tool | What it returns | Live source |
|---|---|---|
| `liquidation_summary` | Market-wide perp liquidations over the last N minutes: long vs short notional, dominant side, top coins — the cascade / short-squeeze signal | Bybit liquidation stream, 24/7 ingester |
| `coin_liquidations` | One coin's liquidations: long vs short notional + the largest single prints | Bybit liquidation stream |
| `funding_rates` | Current perp funding rates with the most-stretched rates surfaced (crowding / carry signal); optional per-coin filter | Bybit tickers |
| `perp_positioning` | Whether leveraged positioning is BUILDING or UNWINDING: open-interest change + funding crowding + which side is getting liquidated, with a plain-language read | Bybit OI/funding snapshots + liquidation stream |
| `best_stable_yield` | Safe single-asset stablecoin lending yields (reputable audited protocols, >$50M TVL), emission-discounted ranking, honest hold-vs-deploy verdict vs the T-bill floor | DefiLlama |
| `convert_currency` | Fiat FX conversion at the live ECB mid-market rate — deterministic arithmetic LLMs get wrong | ECB reference rates |

Liquidation and positioning tools query a rolling 24-hour buffer fed by always-on ingesters; `window_minutes` is capped to that buffer.

## Quick start

### Claude Code
```bash
claude mcp add --transport http jarvis-market-signals https://jarvis.tail12fa6b.ts.net:8443/mcp --header "Authorization: Bearer YOUR_KEY"
```

### Cursor / generic MCP client config
```json
{
  "mcpServers": {
    "jarvis-market-signals": {
      "url": "https://jarvis.tail12fa6b.ts.net:8443/mcp",
      "headers": { "Authorization": "Bearer YOUR_KEY" }
    }
  }
}
```

### curl (no key needed for discovery)
```bash
curl -s -X POST https://jarvis.tail12fa6b.ts.net:8443/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

`tools/call` requires `Authorization: Bearer <key>` (or `x-api-key`). Without one you get a plain JSON 401.

## Auth and limits

- **Anonymous:** `initialize`, `tools/list` — open for discovery.
- **Keyed:** `tools/call` — `Authorization: Bearer jvk_…`, free tier 100 calls/day (rolling 24h), HTTP 429 over quota.
- No OAuth; keys are issued manually via GitHub issue while the service is small.

## Notes

- Market data is informational only — **not financial advice**. Yield figures are estimates from public feeds; verify before acting.
- The server runs on dedicated home infrastructure behind a Tailscale Funnel TLS relay; brief maintenance windows can occur.
- Source for the gateway lives in a private repo; this repo is the public listing, key-request channel, and issue tracker.
