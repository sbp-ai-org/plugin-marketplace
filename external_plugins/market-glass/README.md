# Market Glass for Grok

Hosted MCP connector for [Market Glass](https://market-glass.com) — live market data, portfolio look-through, watchlists, and the MG Investment Committee via `https://mcpfinance.io/mcp`.

This is the Grok / xAI plugin wrapper. It does **not** vendor Market Glass application source. It only declares the public Streamable HTTP MCP endpoint and a skill that tells the agent how to use it.

## Endpoint

```
https://mcpfinance.io/mcp
```

Canonical display URL (users copy it from **Committee settings → Connectors → Copy URL**). A Cloudflare Worker on `mcpfinance.io` proxies to the Supabase edge function and rewrites discovery so OAuth is self-consistent on the branded host.

Do **not** invent a path. Do **not** paste the raw Supabase function URL into new Grok listings.

## Auth (OAuth — no API key)

Market Glass MCP uses **OAuth 2.0 with dynamic client registration**. Claude, ChatGPT, and **Grok** discover the flow automatically. There is **no client ID, client secret, or `MARKET_GLASS_API_KEY`** to configure for Grok.

1. Install / add the connector pointed at `https://mcpfinance.io/mcp`.
2. Grok follows RFC 9728 protected-resource metadata at
   `https://mcpfinance.io/mcp/.well-known/oauth-protected-resource`
   (it also probes `https://mcpfinance.io/.well-known/oauth-authorization-server`).
3. The user signs in to Market Glass and approves the consent screen.
4. Tokens are **per-user Market Glass JWTs**. Every tool call runs as that user, respects their tier/entitlement gates, and bills their **MGT** balance.

Do **not** ask the user for a Bearer API key. That is not how this connector authenticates.

### Manual / enterprise OAuth only (not Grok)

Platforms that cannot do DCR (e.g. Gemini Enterprise) need explicit endpoints from:

`https://hgsckixeeupuaoznwkfb.supabase.co/auth/v1/.well-known/oauth-authorization-server`

- Authorization: `https://hgsckixeeupuaoznwkfb.supabase.co/auth/v1/oauth/authorize`
- Token: `https://hgsckixeeupuaoznwkfb.supabase.co/auth/v1/oauth/token`
- Static client: Supabase Dashboard → Authentication → OAuth Apps
- PKCE enabled; MCP URL still `https://mcpfinance.io/mcp`

Grok does **not** need those static-client steps.

## Install in Grok

Once listed in the [xAI plugin marketplace](https://github.com/xai-org/plugin-marketplace):

```bash
grok plugin install market-glass
```

Or add a custom connector with URL `https://mcpfinance.io/mcp` and complete the OAuth sign-in.

## What the connector can do

All tools run as the connected Market Glass user.

**Market data:** `get_quote`, `get_fundamentals`, `get_financials`, `get_analyst_targets`, `get_chart_data`, `compare_tickers` (2–6), `valuation_snapshot`, `get_market_overview`, `get_sector_performance`, `get_asset_class_performance`, `get_treasury_yields`, `get_economic_indicator`, `get_economic_calendar`, `get_commodities_fx`, `get_fund_allocation`, `sector_drilldown`.

**User data:** `lookup_portfolio`, `create_portfolio` (create-only), `add_holdings_to_portfolio` (append-only; refused for Plaid-synced), `search_symbol`, `lookup_watchlist`, `add_to_watchlist`, `lookup_alerts`, `create_alert`.

**AI:** `research_web`, `generate_chart_artifact`, `ask_copilot` (modes: `chat` default, `strategist`, `research`, `analyst`, `multimodal`, `conversationalist`), `ask_committee` (`power_mode` default `cool`; `medium`/`hot` only when needed — external clients time out; `super_committee` not available in cool).

## Errors

| Code | Meaning | What to do |
|---|---|---|
| 401 | OAuth token expired | Reconnect the connector (do not invent an API key) |
| 403 | No subscription / committee gate | Resolve in Market Glass, not in Grok |
| 429 | MGT credits exhausted | User tops up / waits in Market Glass |

## Privacy

- Do not paste brokerage credentials, SSNs, or account numbers into chat.
- Synced (Plaid) portfolios are read-only; suggest edits in the brokerage.
- Model portfolios use `weight_pct`, not owned shares.

## License

MIT — see [LICENSE](LICENSE).

## Support

- Product: [market-glass.com](https://market-glass.com)
- Copy URL: Committee settings → Connectors
- Publisher: Something Better Productions, LLC — `admin@sbp-ai.com`
