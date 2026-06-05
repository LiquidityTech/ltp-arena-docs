# Resource Manifest — AI Quantitative Trading Competition

> Everything a participant receives or needs to access for the competition.

---

## 1. Competition Account

Each participant receives a dedicated simulation account on the UAT mock matching environment.

| Item | Value |
|------|-------|
| Portfolio ID | *(issued individually by organizer)* |
| Access Key (`LTP_ACCESS_KEY`) | *(issued individually, distributed via secure channel)* |
| Secret Key (`LTP_SECRET_KEY`) | *(issued individually, distributed via secure channel)* |
| API Host (`LTP_API_HOST`) | *(provided by organizer — **required, no default**)* |
| Initial virtual balance | 10,000 USDT (subject to organizer announcement) |
| Position mode | `BOTH` (hedge mode) by default |
| Real-asset risk | **None.** All orders route to the mock matching engine. |

> Credentials are delivered via the registration channel. Treat them as production secrets — never commit, log, or share them.

---

## 2. Service Endpoints

| Service | URL |
|---------|-----|
| REST API | *(use `LTP_API_HOST` value provided by organizer)* |
| Private WebSocket (trading & account events) | `wss://wss-uat.liquiditytech.com/v1/private` |
| Market Data WebSocket (OPEN API) | `wss://mds-uat.liquiditytech.com/marketdata/v2/public` |

### Market Data WebSocket Rate Limits

| Mode | Max connections / IP | Max symbols / connection |
|------|---------------------|--------------------------|
| Unauthenticated | 5 | 5 |
| Authenticated | 40 | 50 |

> Authenticate with your competition API Key to reach the higher limits.

---

## 3. npm Packages

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

| Skill | Location | Purpose |
|-------|----------|---------|
| `ltp-rapidx-config` | `https://github.com/LiquidityTech/ltp-rapidx-skill` | Install CLI, collect credentials, configure MCP, run self-check |
| `ltp-rapidx-trading` | Same repository | Discover tools, manage reads/writes, run live trade verification |

See `README.md` §Step 1 for per-agent-host install commands.

---

## 5. CLI & MCP Tool Catalogue

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

### Account

| CLI | MCP tool |
|-----|----------|
| `rapidx account overview` | `rapidx/account/overview` |
| `rapidx account balance` | `rapidx/account/balance` |
| `rapidx account set-position-mode` | `rapidx/account/set-position-mode` |

### Orders (preview required for all writes)

| CLI | MCP tool |
|-----|----------|
| `rapidx order place-preview` | `rapidx/order/place-preview` |
| `rapidx order place` | `rapidx/order/place` |
| `rapidx order amend-preview` | `rapidx/order/amend-preview` |
| `rapidx order amend` | `rapidx/order/amend` |
| `rapidx order cancel-preview` | `rapidx/order/cancel-preview` |
| `rapidx order cancel` | `rapidx/order/cancel` |
| `rapidx order get` | `rapidx/order/get` |
| `rapidx order list` | `rapidx/order/list` |
| `rapidx order history` | `rapidx/order/history` |

### Positions (preview required for writes)

| CLI | MCP tool |
|-----|----------|
| `rapidx position list` | `rapidx/position/list` |
| `rapidx position history` | `rapidx/position/history` |
| `rapidx position close` | `rapidx/position/close` |
| `rapidx position set-leverage` | `rapidx/position/set-leverage` |

### Algo Orders (preview required for writes)

| CLI | MCP tool |
|-----|----------|
| `rapidx algo list` | `rapidx/algo/list` |
| `rapidx algo place` | `rapidx/algo/place` |
| `rapidx algo amend` | `rapidx/algo/amend` |
| `rapidx algo cancel` | `rapidx/algo/cancel` |

### Trade Utilities

| CLI | MCP tool |
|-----|----------|
| `rapidx trade preview` | `rapidx/trade/preview` |
| `rapidx trade verify-live` | `rapidx/trade/verify-live` |

Full input/output specifications: [`rapidx-api.md`](./rapidx-api.md).

---

## 6. Symbol Universe

Format: `{EXCHANGE}_{TYPE}_{BASE}_{QUOTE}`

Supported exchanges: `BINANCE` · `OKX` · `EDX`

### Common Symbols

| Symbol | Description |
|--------|-------------|
| `BINANCE_PERP_BTC_USDT` | Binance BTC perpetual futures |
| `BINANCE_PERP_ETH_USDT` | Binance ETH perpetual futures |
| `BINANCE_PERP_BNB_USDT` | Binance BNB perpetual futures |
| `BINANCE_PERP_XRP_USDT` | Binance XRP perpetual futures |
| `BINANCE_PERP_ADA_USDT` | Binance ADA perpetual futures |
| `BINANCE_PERP_DOGE_USDT` | Binance DOGE perpetual futures |
| `OKX_PERP_BTC_USDT` | OKX BTC perpetual futures |
| `OKX_PERP_ETH_USDT` | OKX ETH perpetual futures |
| `EDX_PERP_ETH_USDT` | EDX ETH perpetual futures |

Full list:

```bash
rapidx market get-symbol-info --json
```

> **Check symbol rules before trading**: minimum notional, lot size, tick size, and contract size vary by symbol. Run `rapidx market get-symbol-info --input '{"symbol":"..."}' --json` before sizing any order.

---

## 7. Documentation

| Document | Location |
|----------|----------|
| Participant kit overview (EN) | [`README.md`](./README.md) |
| Participant kit overview (CN) | [`README.zh-CN.md`](./README.zh-CN.md) |
| API reference (EN) | [`rapidx-api.md`](./rapidx-api.md) |
| API reference (CN) | [`rapidx-api.zh-CN.md`](./rapidx-api.zh-CN.md) |
| Resource manifest | This document |
| Legacy REST API reference | [`api-reference.md`](api-reference.md) |
| Online documentation | <https://docs.liquiditytech.com/rapidx/> |

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

## 9. Competition Schedule

| Phase | Activity |
|-------|----------|
| Registration | Receive credentials, install CLI, run self-check |
| Development | Build and test agent against live UAT data |
| Submission cut-off | Freeze agent code + model weights |
| Evaluation window | Frozen agents run on live forward market data |
| Scoring & awards | Multi-metric scoring (return, Sharpe, max drawdown, win rate) |

Exact dates communicated via the organizer's official channel.

---

## 10. Support

| Channel | Contact |
|---------|---------|
| Technical support email | `support@liquiditytech.com` |
| Issue tracker | <https://github.com/liquiditytech/rapidx-cli/issues> |
| Discord / community | *(link provided by organizer at registration)* |
| Status page | <https://status.liquiditytech.com> |

---

## 11. Safety & Compliance Reminders

1. **No real assets at risk** — competition runs on the UAT mock matching engine.
2. **Preview before every write** — all writes require preview; the preview never submits a real order.
3. **`LTP_API_HOST` is required** — missing host returns `RCORE01003`.
4. **State confirmation after every write** — call `rapidx order get` or `rapidx position list`. No blind retries.
5. **Secrets never leak** — keys are masked in all CLI/MCP output, logs, and reports.
6. **Single-track competition** — all participants compete under identical conditions.

---

**中文版**: [resources.zh-CN.md](./resources.zh-CN.md)
