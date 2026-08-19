---
name: market-glass
description: Use the Market Glass hosted MCP (mcpfinance.io/mcp) for live quotes, fundamentals, portfolios, watchlists, and the MG Investment Committee. Trigger when the user asks about tickers, markets, portfolios, or connecting Market Glass to Grok.
---

# Market Glass

Hosted Streamable HTTP MCP at `https://mcpfinance.io/mcp`.

Auth is **OAuth 2.0 + dynamic client registration**. Grok discovers it automatically. There is **no API key**. Tokens are the connected user's Market Glass JWT. Tool calls bill that user's **MGT** balance.

Users copy the URL from **Committee settings → Connectors → Copy URL**.

## Before calling tools

1. Confirm the connector URL is exactly `https://mcpfinance.io/mcp` (do not invent a path; do not use the raw Supabase function URL for new connections).
2. If tools fail with **401**, the OAuth token expired — tell the user to **reconnect** the connector and complete Market Glass sign-in. Do **not** ask for `MARKET_GLASS_API_KEY`, `mg_live_…`, or a Bearer secret.
3. **403** = account lacks access (subscription or committee `min_role` gate). Resolve in Market Glass.
4. **429** = MGT credits exhausted.
5. Never log or echo tokens.

## Tool groups

### Market data

`get_quote`, `get_fundamentals`, `get_financials`, `get_analyst_targets`, `get_chart_data`, `compare_tickers` (2–6 symbols), `valuation_snapshot`, `get_market_overview`, `get_sector_performance`, `get_asset_class_performance`, `get_treasury_yields`, `get_economic_indicator`, `get_economic_calendar`, `get_commodities_fx`, `get_fund_allocation` (ETF **and** mutual fund; mutual funds fall back to latest SEC N-PORT), `sector_drilldown`.

### User data

- `lookup_portfolio` — list or one by name; includes look-through analytics and an `interpretation` string. Every row has `portfolio_type` + `source`.
- `create_portfolio` — create **only**; never overwrite. `portfolio_type: "active" | "model"`. Holdings `asset_type`: `stock`, `etf`, `mutual-fund`, `closed-end-fund`, `bond`, `cash`, `commodity`, `reit`, `crypto` (omit to auto-detect).
- `add_holdings_to_portfolio` — append-only. Refused for Plaid-synced. Models take `weight_pct`; active take `shares`.
- `search_symbol`, `lookup_watchlist`, `add_to_watchlist`, `lookup_alerts`, `create_alert`.

Portfolio kinds:

| Kind | Identified by | Holdings mean |
|---|---|---|
| Active (manual) | `portfolio_type: "active"`, `source: "manual"` | Real `shares` × price |
| Model | `portfolio_type: "model"` | Target `weight_pct` (~100). No owned dollars |
| Synced brokerage | `portfolio_type: "active"`, `source: "plaid"` | Read-only Plaid mirror |

Never treat model weights as owned shares. Never compute dollar values from model holdings. Never try to edit a Plaid-synced portfolio via MCP. Combining portfolios is **in-app only**, not via MCP.

### AI

- `research_web` — live web research with citations
- `generate_chart_artifact` — stored chart image URL
- `ask_copilot` — one-shot; modes `chat` (default), `strategist`, `research`, `analyst`, `multimodal`, `conversationalist`; optional `context_ticker`. Prefer over committee for single-perspective questions.
- `ask_committee` — multi-agent verdict. `power_mode`: `cool` (default, ~30–60s — use this for Grok; clients time out), `medium` (~2min), `hot` (~4min). `super_committee: true` = multi-round debate, **not** available in cool. Access gated by `committee_settings.min_role` (commonly `platinum`).

## How to answer

- Prefer MCP tools over training-data prices.
- Cite ticker + tool result; do not invent quotes or holdings.
- If unauthorized, say so from the HTTP status — do not fabricate portfolio or committee data.
- Do not invent client PII or unstated Market Glass features.

## Privacy

Do not ask for or store brokerage passwords, SSNs, or raw account numbers. Point users to Market Glass / Plaid for account linking.
