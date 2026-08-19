---
name: market-glass
description: Use the Market Glass hosted MCP (mcpfinance.io/mcp) for live quotes, fundamentals, portfolios, watchlists, and investment research. Trigger when the user asks about tickers, markets, portfolios, or connecting Market Glass to Grok.
---

# Market Glass

Hosted Streamable HTTP MCP at `https://mcpfinance.io/mcp`. Auth is `Authorization: Bearer <MARKET_GLASS_API_KEY>` from https://market-glass.com/settings/mcp.

## Before calling tools

1. Confirm the Grok connector is pointed at `https://mcpfinance.io/mcp` (not a guessed path).
2. If tools fail with 401, ask the user to create a key at Settings → MCP and set `MARKET_GLASS_API_KEY`. Do not invent a key.
3. Personal keys (`mg_live_…`) unlock user-data tools. Organization keys (`mgo_live_…`) are market-data only.
4. Never log, echo, or commit the API key.

## Tool groups

### Market data (any live key)

Use these for public-market questions:

- `get_quote`, `get_fundamentals`, `get_financials`, `get_analyst_targets`, `get_chart_data`
- `compare_tickers` (2–6 symbols), `valuation_snapshot`
- `get_market_overview`, `get_sector_performance`, `get_asset_class_performance`
- `get_treasury_yields`, `get_economic_indicator`, `get_economic_calendar`, `get_commodities_fx`
- `get_fund_allocation`, `sector_drilldown`

### User data (Personal key)

- `lookup_portfolio` — list portfolios or one by name, with look-through analytics
- `create_portfolio` — create only; never overwrite an existing portfolio
- `add_holdings_to_portfolio` — append-only; refused for Plaid-synced portfolios
- `search_symbol`, `lookup_watchlist`, `add_to_watchlist`
- `lookup_alerts`, `create_alert`

Portfolio kinds:

| Kind | How to tell | Holdings mean |
|---|---|---|
| Active (manual) | `portfolio_type: "active"`, `source: "manual"` | Real `shares` |
| Model | `portfolio_type: "model"` | Target `weight_pct`, not owned dollars |
| Synced brokerage | `portfolio_type: "active"`, `source: "plaid"` | Read-only Plaid mirror |

Never treat model weights as owned shares. Never try to edit a Plaid-synced portfolio via MCP.

### AI (Personal key + plan)

- `research_web`, `generate_chart_artifact`, `ask_copilot`, `ask_committee`

## How to answer

- Prefer MCP tools over training-data prices.
- Cite the ticker and the tool result; do not invent quotes or holdings.
- If a user-data tool is unauthorized, say the user needs a Personal key — do not fall back to fabricated portfolio data.
- Do not invent client PII or unstated Market Glass features.

## Privacy

Do not ask for or store brokerage passwords, SSNs, or raw account numbers. Point users to Market Glass / Plaid for account linking.
