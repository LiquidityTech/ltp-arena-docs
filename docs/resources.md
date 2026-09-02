# Resource Manifest — AI Quantitative Trading Competition

> Everything a participant receives or needs to access for the competition.

---

## 1. Competition Account

Each participant receives a dedicated account. Log in at **https://www.liquiditytech.com/** (Phase II Mainnet / Track B RapidX) or **https://uat.liquiditytech.com/** (Phase I Sandbox) and generate your API credentials under **Profile → API Management → Generate**.

| Item | Value |
|------|-------|
| Login URL | Phase I (Sandbox): `https://uat.liquiditytech.com/` · Phase II (Mainnet) / Track B RapidX: `https://www.liquiditytech.com/` |
| API Key creation | Profile → API Management → Generate |
| Portfolio ID | *(issued individually by organizer)* |
| Access Key (`LTP_ACCESS_KEY`) | Generated under API Management |
| Secret Key (`LTP_SECRET_KEY`) | Generated under API Management |
| API Host (`LTP_API_HOST`) | Phase I (Sandbox): `https://api.ltp-contest.com` · Phase II (Mainnet): `https://api.liquiditytech.com` |
| Initial capital | 1,000 USDT (subject to organizer announcement) |
| Position mode | `BOTH` (hedge mode) by default |
| Real-asset risk | **None.** All orders route to the simulation matching engine. |

**API Key permissions:**

| Permission | Description |
|------------|-------------|
| Read | View balances, positions, and order history |
| Trade (RapidX) | Place and manage orders on RapidX |

> **Phase II Onboarding**: During the preparation phase, teams provide their deposit address to the Organizing Committee, who will fund the account with 1,000 USDT initial capital on their behalf.

> **Account Restrictions**: Once the main competition begins, self-service deposits and withdrawals are disabled — the deposit API is banned. Transfers between Binance and OKX within RapidX are permitted, as all teams trade exclusively on RapidX. This applies to both Track A and Track B.

> Treat your credentials as production secrets — never commit, log, or share them.

---

## 2. Service Endpoints

See [`03-advanced-api.md`](./03-advanced-api.md) for authentication, signature algorithm, and full endpoint reference.

| Service | Phase I (Sandbox) | Phase II (Mainnet) |
|---------|-------------------|--------------------|
| Trading REST API | `https://api.ltp-contest.com` | `https://api.liquiditytech.com` |
| News Feed REST API | `https://api.ltp-contest.com` | `https://api.liquiditytech.com` |
| Market Data WebSocket | `wss://md.ltp-contest.com/marketdata/v2/public` | `wss://md.liquiditytech.com/marketdata/v2/public` |
| User Data WebSocket | `wss://wss.ltp-contest.com/v1/private` | `wss://wss.liquiditytech.com/v1/private` |
| News Feed WebSocket | `wss://feeds.ltp-contest.com/feeds/v2/public` | `wss://feeds.ltp-contest.com/feeds/v2/public` (unchanged across phases) |
| Leaderboard UI | `https://arena-uat.liquiditytech.com/leaderboard` | `https://arena.liquiditytech.com/leaderboard` |

> **Host selection**: Use the Phase I (Sandbox) hosts during the sandbox/development period and the Phase II (Mainnet) hosts once the main competition begins. The Phase II REST API at `https://api.liquiditytech.com` is consistent with the official production API documentation.

> For programmatic access, see [Leaderboard API](../advanced-api-docs/leaderboard.md).

### Market Data WebSocket Rate Limits

| | Unauthenticated | Authenticated |
|---|---|---|
| Max connections per IP | 5 | 40 |
| Max trading pairs per connection | 5 | 50 |

> Authenticate with your competition API Key to reach the higher limits. Limits are counted by **trading pair**, not by channel — subscribing to BBO + TICKER + TRADE for the same symbol counts as 1 pair.

### News Feed WebSocket Connection Limits

| Limit | Value | Notes |
|-------|-------|-------|
| Max concurrent connections / IP | **5** | Exceeding this returns HTTP **429** — handshake rejected |
| Heartbeat timeout | **90 seconds** | Server closes the connection if no `ping` is received within 90 s |
| Heartbeat check cycle | 30 seconds | Server scans for stale connections every 30 s |

> **Reconnection note**: subscription state is not preserved on the server. After a reconnect, you must re-send the `subscribe` message for all channels.

---

## 3. npm Packages

See [`01-quickstart.md`](./01-quickstart.md) §Step 1 for install and registry troubleshooting.

| Package | Description | Install |
|---------|-------------|---------|
| `@liquiditytech/rapidx-cli` | RapidX CLI (`rapidx` command) + `rapidx mcp serve` | `npm install -g @liquiditytech/rapidx-cli` |

Run `rapidx --version` after install to confirm the version.

If npm maps `@liquiditytech` to a custom registry, point it to the official registry first:

```bash
npm config set @liquiditytech:registry https://registry.npmjs.org/
npm install -g @liquiditytech/rapidx-cli@latest
```

---

## 4. Skills

See [`01-quickstart.md`](./01-quickstart.md) §Step 3 Path A for per-agent-host install commands (Claude Code, Codex, Cursor, OpenClaw, Hermes).

| Skill | Location | Purpose |
|-------|----------|---------|
| `ltp-rapidx-config` | `https://github.com/LiquidityTech/ltp-rapidx-skill` | Install CLI, collect credentials, configure MCP, run self-check |
| `ltp-rapidx-trading` | Same repository | Discover tools, manage reads/writes, run live trade verification |

---

## 5. CLI & MCP Tool Catalogue

See [`02-cli-mcp-reference.md`](./02-cli-mcp-reference.md) for full input/output specs, the preview-then-submit pattern, and automation mode.

All CLI commands use the format `rapidx <domain> <action> --input '<json>' --json`.  
All MCP tools use the `rapidx/` prefix.

### Discovery & Diagnostics

| CLI | MCP tool | Description |
|-----|----------|-------------|
| `rapidx schema --json` | `rapidx/tools` | List all capabilities and schemas |
| `rapidx self-check --read-only --json` | `rapidx/self-check` | Verify read-only connectivity |
| `rapidx doctor --json` | CLI only | Local diagnostics (version, credential source, invocation mode) |
| `rapidx auth check` | CLI only | Credential check without printing secrets |
| `rapidx invocation check` | CLI only | Verify the invocation style is supported |
| `rapidx update check --json` | `rapidx/update/check` | Check for CLI/skills updates |

### Market Data

| CLI | MCP tool |
|-----|----------|
| `rapidx market get-ticker` | `rapidx/market/get-ticker` |
| `rapidx market get-orderbook` | `rapidx/market/get-orderbook` |
| `rapidx market get-klines` | `rapidx/market/get-klines` |
| `rapidx market get-funding-rate` | `rapidx/market/get-funding-rate` |
| `rapidx market get-mark-price` | `rapidx/market/get-mark-price` |
| `rapidx market get-symbol-info` | `rapidx/market/get-symbol-info` |
| `rapidx market get-open-interest` | `rapidx/market/get-open-interest` |

### Portfolio

| CLI | MCP tool |
|-----|----------|
| `rapidx portfolio overview` | `rapidx/portfolio/overview` |
| `rapidx portfolio assets` | `rapidx/portfolio/assets` |
| `rapidx portfolio statement` | `rapidx/portfolio/statement` |
| `rapidx portfolio user-fee-rate` | `rapidx/portfolio/user-fee-rate` |
| `rapidx portfolio position-bracket` | `rapidx/portfolio/position-bracket` |
| `rapidx portfolio set-position-mode` | `rapidx/portfolio/set-position-mode` |

### Orders (preview required for all writes)

| CLI | MCP tool |
|-----|----------|
| `rapidx order place-preview` | `rapidx/order/place-preview` |
| `rapidx order place` | `rapidx/order/place` |
| `rapidx order replace-preview` | `rapidx/order/replace-preview` |
| `rapidx order replace` | `rapidx/order/replace` |
| `rapidx order cancel-preview` | `rapidx/order/cancel-preview` |
| `rapidx order cancel` | `rapidx/order/cancel` |
| `rapidx order cancel-all` | `rapidx/order/cancel-all` |
| `rapidx order query` | `rapidx/order/query` |
| `rapidx order open-orders` | `rapidx/order/open-orders` |
| `rapidx order history` | `rapidx/order/history` |

### Transactions

| CLI | MCP tool |
|-----|----------|
| `rapidx transaction executions` | `rapidx/transaction/executions` |

### Positions (preview required for writes)

| CLI | MCP tool |
|-----|----------|
| `rapidx position query` | `rapidx/position/query` |
| `rapidx position history` | `rapidx/position/history` |
| `rapidx position get-leverage` | `rapidx/position/get-leverage` |
| `rapidx position set-leverage` | `rapidx/position/set-leverage` |
| `rapidx position close` | `rapidx/position/close` |
| `rapidx position close-all` | `rapidx/position/close-all` |

### Algo Orders (preview required for writes)

| CLI | MCP tool |
|-----|----------|
| `rapidx algo place` | `rapidx/algo/place` |
| `rapidx algo replace` | `rapidx/algo/replace` |
| `rapidx algo cancel` | `rapidx/algo/cancel` |
| `rapidx algo open-orders` | `rapidx/algo/open-orders` |
| `rapidx algo history` | `rapidx/algo/history` |
| `rapidx algo query` | `rapidx/algo/query` |

### Automation Sessions

| CLI | MCP tool |
|-----|----------|
| `rapidx automation start` | `rapidx/automation/start` |
| `rapidx automation list` | `rapidx/automation/list` |
| `rapidx automation status` | `rapidx/automation/status` |
| `rapidx automation extend` | `rapidx/automation/extend` |
| `rapidx automation stop` | `rapidx/automation/stop` |

### Trade Utilities

| CLI | MCP tool |
|-----|----------|
| `rapidx trade preview` | `rapidx/trade/preview` |
| `rapidx trade verify-live` | `rapidx/trade/verify-live` |

Full input/output specifications: [`02-cli-mcp-reference.md`](./02-cli-mcp-reference.md).

---

## 6. Symbol Universe

### Track A — Tradable Pairs Whitelist (Phase I)

> **Glossary**: **PERP** = perpetual futures (a derivative that tracks an asset price with no expiry). **USDT-margined** = margin and PnL are settled in USDT (not the underlying asset).

- **50 pairs**, USDT-margined Binance perpetuals, format: `BINANCE_PERP_{BASE}_USDT`
- Fixed for the competition period; not dynamically adjusted by market cap
- Selection criteria: Binance USDT perp ✓ | System whitelist ✓ | Listed ≥ 6 months (CoinGecko) ✓

> **Check symbol rules before trading**: minimum notional, lot size, tick size, and contract size vary by symbol. Run `rapidx market get-symbol-info --input '{"symbol":"..."}' --json` before sizing any order.

| # | Symbol | Asset Name | Launch Date | Market Cap (B USD) |
|---|--------|------------|-------------|-------------------|
| 1 | `BINANCE_PERP_BTC_USDT` | Bitcoin | 2009-01-03 | 12,372 |
| 2 | `BINANCE_PERP_ETH_USDT` | Ethereum | 2015-07-30 | 2,072 |
| 3 | `BINANCE_PERP_BNB_USDT` | BNB | 2017-07-08 | 758 |
| 4 | `BINANCE_PERP_USDC_USDT` | USD Coin | — | 732 |
| 5 | `BINANCE_PERP_XRP_USDT` | XRP | 2013-01-01 | 686 |
| 6 | `BINANCE_PERP_SOL_USDT` | Solana | 2020-04-10 | 472 |
| 7 | `BINANCE_PERP_TRX_USDT` | TRON | 2017-09-13 | 301 |
| 8 | `BINANCE_PERP_HYPE_USDT` | Hyperliquid | 2024-11-29 | 150 |
| 9 | `BINANCE_PERP_DOGE_USDT` | Dogecoin | 2013-12-06 | 116 |
| 10 | `BINANCE_PERP_ZEC_USDT` | Zcash | 2016-10-28 | 73 |
| 11 | `BINANCE_PERP_XLM_USDT` | Stellar | 2014-07-31 | 68 |
| 12 | `BINANCE_PERP_ADA_USDT` | Cardano | 2017-09-29 | 61 |
| 13 | `BINANCE_PERP_XMR_USDT` | Monero | 2014-04-18 | 60 |
| 14 | `BINANCE_PERP_LINK_USDT` | Chainlink | 2017-09-19 | 58 |
| 15 | `BINANCE_PERP_CC_USDT` | Canton Network | 2025-12-06 | 54 |
| 16 | `BINANCE_PERP_BCH_USDT` | Bitcoin Cash | 2017-08-01 | 45 |
| 17 | `BINANCE_PERP_LTC_USDT` | Litecoin | 2011-10-07 | 34 |
| 18 | `BINANCE_PERP_HBAR_USDT` | Hedera | 2019-09-16 | 31 |
| 19 | `BINANCE_PERP_SUI_USDT` | Sui | 2023-05-03 | 30 |
| 20 | `BINANCE_PERP_LAB_USDT` | LAB | 2025-12-01 | 30 |
| 21 | `BINANCE_PERP_AVAX_USDT` | Avalanche | 2020-09-22 | 30 |
| 22 | `BINANCE_PERP_XAUT_USDT` | Tether Gold | 2020-01-24 | 25 |
| 23 | `BINANCE_PERP_NEAR_USDT` | NEAR Protocol | 2020-04-22 | 25 |
| 24 | `BINANCE_PERP_1000SHIB_USDT` | Shiba Inu | 2020-08-01 | 25 |
| 25 | `BINANCE_PERP_M_USDT` | MemeCore | 2025-07-03 | 23 |
| 26 | `BINANCE_PERP_TAO_USDT` | Bittensor | 2023-11-01 | 21 |
| 27 | `BINANCE_PERP_UNI_USDT` | Uniswap | 2020-09-16 | 20 |
| 28 | `BINANCE_PERP_PAXG_USDT` | PAX Gold | 2019-09-25 | 19 |
| 29 | `BINANCE_PERP_WLFI_USDT` | World Liberty Financial | 2025-09-01 | 18 |
| 30 | `BINANCE_PERP_ASTER_USDT` | Aster | 2025-09-17 | 17 |
| 31 | `BINANCE_PERP_ONDO_USDT` | Ondo Finance | 2024-01-18 | 16 |
| 32 | `BINANCE_PERP_WLD_USDT` | Worldcoin | 2023-07-24 | 15 |
| 33 | `BINANCE_PERP_DOT_USDT` | Polkadot | 2020-08-19 | 14 |
| 34 | `BINANCE_PERP_SKY_USDT` | Sky (MakerDAO) | 2018-01-29 | 14 |
| 35 | `BINANCE_PERP_AAVE_USDT` | Aave | 2017-11-04 | 13 |
| 36 | `BINANCE_PERP_MORPHO_USDT` | Morpho | 2022-08-01 | 13 |
| 37 | `BINANCE_PERP_ICP_USDT` | Internet Computer | 2021-05-10 | 12 |
| 38 | `BINANCE_PERP_ETC_USDT` | Ethereum Classic | 2016-07-20 | 11 |
| 39 | `BINANCE_PERP_DEXE_USDT` | DeXe | 2020-10-28 | 11 |
| 40 | `BINANCE_PERP_1000PEPE_USDT` | Pepe | 2023-04-17 | 10 |
| 41 | `BINANCE_PERP_QNT_USDT` | Quant | 2018-08-10 | 10 |
| 42 | `BINANCE_PERP_BEAT_USDT` | Audiera | 2025-11-01 | 9 |
| 43 | `BINANCE_PERP_KAS_USDT` | Kaspa | 2021-11-07 | 9 |
| 44 | `BINANCE_PERP_STABLE_USDT` | Stable | 2025-12-23 | 8 |
| 45 | `BINANCE_PERP_RENDER_USDT` | Render | 2020-10-01 | 8 |
| 46 | `BINANCE_PERP_ATOM_USDT` | Cosmos | 2019-03-14 | 8 |
| 47 | `BINANCE_PERP_JUP_USDT` | Jupiter | 2024-01-31 | 8 |
| 48 | `BINANCE_PERP_POL_USDT` | Polygon | 2019-04-26 | 8 |
| 49 | `BINANCE_PERP_ALGO_USDT` | Algorand | 2019-06-19 | 8 |
| 50 | `BINANCE_PERP_JST_USDT` | JUST | 2020-05-05 | 8 |

### Rate Limits

| Endpoint | Limit |
|----------|-------|
| Place Order | 1 req / 5s |
| Cancel Order | 1 req / 5s |
| Replace Order | 1 req / 5s |
| GET `/api/v1/market/fundingRate` | 3 req / 10s |
| GET `/api/v1/market/markPrice` | 3 req / 10s |
| GET `/api/v1/trading/sym/info` | 3 req / 10s |
| DELETE `/api/v1/trading/positions` | 1 req / 10s |
| All other endpoints | See individual endpoint docs |

---

## 7. Documentation

| Document | Location |
|----------|----------|
| Getting started | [`01-quickstart.md`](./01-quickstart.md) |
| CLI & MCP reference | [`02-cli-mcp-reference.md`](./02-cli-mcp-reference.md) |
| Advanced REST & WebSocket | [`03-advanced-api.md`](./03-advanced-api.md) |
| News Feed API (REST + WebSocket) | [`04-news-feed.md`](./04-news-feed.md) |
| Resource manifest | This document |
| GitHub repository | <https://github.com/LiquidityTech/ltp-arena-docs> |

---

## 8. Verification

Use the built-in CLI self-check — no extra dependencies required:

```bash
rapidx auth check
rapidx self-check --read-only --json
```

Each check reports **PASS** / **EXPECTED_ERROR** / **FAIL** / **NOT_VERIFIED**.

For an end-to-end live trade test (submits a real post-only order then cancels):

```bash
rapidx trade verify-live --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "maxNotional": "10",
  "clientOrderId": "verify-live-001",
  "explicitUserConsent": true,
  "acceptedRiskText": "I authorize a real verification order for BINANCE_PERP_BTC_USDT BUY maxNotional 10 with cancel cleanup."
}' --json
```

---

## 9. Competition Structure & Schedule

### Schedule & Timeline

| Date | Milestone |
|------|-----------|
| June 15 | Early Bird Registration |
| July 20 | Track A Phase I officially commences |
| Aug 21 | Track A Phase I ends (23:59 GMT+8) |
| Aug 24 | Reasoning Log submission deadline (23:59 GMT+8) |
| Aug 28 | Phase II advancement announcement (before, GMT+8) |
| Sep 7 | Dual-track main competition begins |
| Oct 31 | Arena closes — competition ends |
| Nov 15 | Official results announcement |

### Track A: Logic Frontier

Track A runs in two phases. **Phase I scores do NOT carry over to Phase II.** Phase II resets all teams' NAV; LTP allocates **1,000 USDT live seed capital** per team. The **top 30 teams** from Phase I advance.

**Phase II — Mainnet Duel Finals**

- **Tradable instruments:** Binance OR OKX perpetual swaps (Phase I was Binance only)
- **Leverage limit:** max 5x (Phase I: 2x)
- **Elimination trigger:** Equity < 800 USDT → forced liquidation + immediate elimination
- **Scoring:** ROI (20%) + PnL (25%) + Sharpe Ratio (40%) + MDD (15%) + AI Agent Reference Metrics (AI-Adjusted PnL, AI Engagement)
- **Core evaluation:** institutional-grade trading performance + Reasoning Log logical quality

### Track B: Liquidity Pro

> **Account Restrictions (applies to all Track B participants)**: Once the main competition begins, self-service deposits and withdrawals are disabled — the deposit API is banned. Transfers between Binance and OKX within RapidX are permitted, as all teams trade exclusively on RapidX. See [`§1`](#1-competition-account) for the Phase II onboarding funding flow.

For quant hedge funds and professional trading teams. Focus: HFT execution quality, multi-factor alpha discovery, ML/DL alpha capture in deep liquidity.

- **Scoring:** Sharpe Ratio (45%) + PnL (30%) + Scaling Capacity (10%) + MDD (15%)
- **Strategy-agnostic;** unified leaderboard (arbitrage, trend-following, CTA, HFT, market-making, AI Agent-driven)

**Participation Paths**

| Path | API Requirement | Trading Frequency | Fees |
|------|----------------|-------------------|------|
| A — External System | Read-Only API Keys from LTP-supported exchanges (Binance, OKX, Bybit, Coinbase, Deribit, CME, SGX, prediction markets) | No upper limit; HFT supported | Follow selected venue's fee schedule |
| B — LTP Infrastructure | RapidTrade / RapidX / RapidMarket 2.0 | Millisecond-level execution | Standard DMA / RapidX fees |

- **Asset classes:** Spot, Perpetual Futures, Delivery Futures, Options, OTC. Profits from low-liquidity or abnormal-volatility assets are excluded from the final score.
- **Capital efficiency (Path B):** unified account, shared margin across CEX/DEX/OTC/traditional exchanges, cross-market settlement.

**Scaling Effect Tiers (Track B)**

| Tier | Capital Range (USDT) |
|------|---------------------|
| 1 | 10,000 – 100,000 |
| 2 | 100,000 – 1,000,000 |
| 3 | 1,000,000+ |

**Fee Policy**

- **Clearing Fee Minimum Charge:** waived for all teams.
- **Track A:** VIP 5-level trading fees.

---

## 10. Scoring Methodology

### Sharpe Ratio

- **Sampling:** daily snapshots (not hourly)
- `daily_return[d] = NAV[UTC 23:00] / NAV[UTC 00:00] − 1`
- Only fully completed days are counted; the current in-progress day is excluded
- Recalculated at 00:00 UTC daily; NULL until ≥ 2 complete days
- **Formula:** `Sharpe = AVERAGE(daily_returns) / STDEV(daily_returns) × √365`
- **Risk-free rate:** 0; **annualization:** √365 (calendar days)
- Sharpe may be negative even when PnL/ROI is positive (inactive/loss days drag the average)

### Cross-Team Normalization

- The leaderboard displays the **raw** Sharpe
- **Composite ranking:** Z-Score → CDF percentile within the eligible pool (non-eliminated teams), then weighted
- **Track A composite** = `0.40 × S(Sharpe) + 0.25 × S(PnL) + 0.20 × S(ROI) + 0.15 × MDD Tier Score`

### Elimination Recomputation

- **Forward-only** from the elimination point, NOT retroactive
- An eliminated team's pre-elimination scoring data remains valid; subsequent performance is excluded from the eligible pool
- Remaining teams are recomputed against the reduced pool

### MDD (Max Drawdown)

- **Formula:** `MAX((running_peak[t] − nav[t]) / running_peak[t])` across all `t`
- 24 hourly snapshots/day (00:00–23:00 UTC)
- **Monotonically non-decreasing** — does not recover even if NAV bounces back
- The system uses minute/tick-level equity internally; ~0.05% deviation from the hourly calculation is expected

---

## 11. Leaderboard

- **URL:** https://arena.liquiditytech.com/leaderboard
- **Note:** some statistics require a full completed trading day (T+1) to compute accurately; early data may still be catching up

**Track A columns**

| Column | Description |
|--------|-------------|
| # | Leaderboard position |
| Team Name | Registered team name |
| Score | Weighted composite score |
| Ann. Return | Annualized return |
| Return % | Cumulative return percentage |
| PnL | Profit and loss |
| Sharpe | Risk-adjusted return (see §10) |
| MDD | Maximum drawdown |
| Total Trades | Number of trades executed |
| AI-Adj. PnL | AI-adjusted PnL (AI Agent reference metric) |
| AI Engagement | AI engagement metric (AI Agent reference metric) |
| Equity Curve | Equity curve chart |

**Track B columns**

| Column | Description |
|--------|-------------|
| Rank | Leaderboard position |
| Team Name | Registered team name |
| Overall Score | Weighted composite score |
| PnL | Profit and loss |
| Sharpe Ratio | Risk-adjusted return (see §10) |
| Total Trades | Number of trades executed |
| MDD | 15–20% → yellow; >20% → red |
| Total Transactions | Number of transactions |
| Strategy | Declared strategy type |
| State | 🟢 Live / ⚠️ Warning / ❌ Eliminated |

---

## 12. AI API & Token Usage (Track A)

- **LLM Model:** MiniMax M3
- **Daily budget:** $10 USD per team per day; unused tokens expire at end of day with no rollover; the next day re-issues $10
- **Cache TTL:** 5 minutes — use caching to control cost

**Pricing Table**

| Input Token Length | Price Type | Unit Price (USD / million tokens) |
|--------------------|-----------|----------------------------------|
| ≤ 512k | Input | 0.31 |
| ≤ 512k | Output | 1.24 |
| ≤ 512k | Cache Read | 0.06 |
| > 512k | Input | 0.62 |
| > 512k | Output | 2.47 |
| > 512k | Cache Read | 0.12 |

**Compliance Rules**

- Teams must **exclusively use the Organizer-provided AI API** throughout the competition. Third-party or self-hosted AI APIs in the trading decision chain are **prohibited** → immediate disqualification + invalidation of all results.
- Compliance review covers **ALL** usage levels (no de minimis threshold; negligible-but-nonzero usage is also reviewed).
- **AI must be the primary decision driver** — processing unstructured info (news, policy, social sentiment) and producing substantiated trading decisions. Using AI merely as a pre-trade gate check or logger does **NOT** qualify.
- Development tools (Cursor / ChatGPT / GitHub Copilot) for coding/debugging are **NOT** restricted.

### Query AI Usage

```
GET https://ai.ltp-contest.com/key/info
Authorization: Bearer <your-ai-api-key>
```

Returns the cumulative USD spend for the current AI API Key.

```bash
# Check AI usage before running agent
curl -s https://ai.ltp-contest.com/key/info \
  -H "Authorization: Bearer <your-ai-api-key>"
```

> **Best practice:** after validating your strategy logic, run your MCP-based agent as a **background scheduled script** rather than interactively. This avoids redundant token usage from repeated manual invocations.

---

## 13. Reasoning Log (Track A)

- Submit **after Track A Phase I ends**; covers **every** order/trade operation (place, cancel, open, close) with the Agent's full reasoning, decision basis, and result
- **Submission deadline:** Aug 24, 23:59 (GMT+8) — send to **events@liquiditytech.com**
- **Failure to submit may affect your team's final eligibility to advance** to the next stage
- **Historical gaps:** submit as-is and note the gap (state when complete model reasoning was introduced; prior ops relied on rule-based logic) — do **NOT** retroactively reconstruct
- **Exchange-triggered exits** (TPSL auto-fill): reasoning is required for the **placement decision**, NOT for the auto-execution
- **Granularity:** same-strategy batch trades may share one reasoning record; AI must continuously participate as the decision/explanatory agent throughout the competition — not a one-time intervention

---

## 14. Support

For competition support, reach out via:
- **Telegram**: Liquidity Arena #1
- **Email**: *(contact the organizing team)*

---

## 15. Safety & Compliance Reminders

1. **No real assets at risk (Phase I)** — Phase I runs on the competition simulation environment with virtual capital.
2. **Elimination threshold** — Elimination is triggered when **Equity < 800 USDT**.
3. **Agent uptime ≥ 90%** — uptime below 90% results in disqualification; 90–95% incurs graduated score penalties.
4. **Preview before every write** — all writes require preview; the preview never submits a real order.
5. **`LTP_API_HOST` is required** — missing host returns `RCORE01003`.
6. **State confirmation after every write** — call `rapidx order query` or `rapidx position query`. No blind retries.
7. **Secrets never leak** — keys are masked in all CLI/MCP output, logs, and reports.

