# Market Glass for Grok

Hosted MCP connector for [Market Glass](https://market-glass.com) — live market data, portfolio look-through, watchlists, and investment research via `https://mcpfinance.io/mcp`.

This is the Grok / xAI plugin wrapper. It does **not** vendor Market Glass application source. It only declares the public Streamable HTTP MCP endpoint and a skill that tells the agent how to use it.

## Install in Grok

Once this plugin is listed in the [xAI plugin marketplace](https://github.com/xai-org/plugin-marketplace):

```bash
grok plugin install market-glass
```

Or add the MCP server by hand:

- **URL:** `https://mcpfinance.io/mcp`
- **Auth:** `Authorization: Bearer <MARKET_GLASS_API_KEY>`

## API key

1. Sign in at [market-glass.com](https://market-glass.com).
2. Open **Settings → MCP** (`https://market-glass.com/settings/mcp`).
3. Create a **Personal** key for portfolio / watchlist / alert tools, or an **Organization** key for market-data-only use.
4. Set `MARKET_GLASS_API_KEY` in the Grok connector. Never commit the key.

Keys are `mg_live_…` (Personal) or `mgo_live_…` (Organization). Both use the same `Authorization: Bearer` header.

## What the connector can do

Market data (any live key):

- Quotes, fundamentals, financials, analyst targets, charts
- Ticker compare and valuation snapshot
- Market overview, sector / asset-class performance
- Treasury yields, economic indicators / calendar, commodities + FX
- Fund / ETF allocation look-through

User data (Personal key):

- Lookup / create portfolios and append holdings
- Search symbols, watchlists, alerts

AI (Personal key + plan entitlements):

- Web research, chart artifacts, copilot, investment committee

## Privacy

- Do not paste brokerage credentials, Social Security numbers, or account numbers into chat.
- Synced (Plaid) portfolios are read-only; suggest edits in the brokerage, not via MCP.
- Model portfolios use `weight_pct`, not owned shares.

## License

MIT — see [LICENSE](LICENSE).

## Support

- Product: [market-glass.com](https://market-glass.com)
- Publisher: Something Better Productions, LLC — `admin@sbp-ai.com`
