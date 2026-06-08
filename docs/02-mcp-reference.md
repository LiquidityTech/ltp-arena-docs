# RapidX CLI & MCP Reference

> Complete tool reference for all 34 MCP tools and 41 CLI capabilities.

The CLI and MCP server share the same core executor. `rapidx mcp serve` exposes the same capabilities as MCP tools — no parallel implementations.

---

## Response Envelope

Every CLI command and MCP tool returns:

```json
{
  "ok": true,
  "status": "PASS",
  "data": {},
  "auditId": "optional-audit-id",
  "evidence": [
    {
      "source": "real_tool_call",
      "toolOrCommandEvidence": "rapidx market get-ticker",
      "timestamp": "2026-06-08T00:00:00.000Z"
    }
  ]
}
```

Error responses may include a `details` object: `upstreamStatus`, `upstreamCode`, `upstreamMessage`, `hint`, `attempts`.

| Status | Meaning |
|--------|---------|
| `PASS` | Completed successfully |
| `FAIL` | Failed — local, upstream, or unexpected |
| `BLOCKED` | Blocked by local validation (e.g. missing preview token) |
| `NOT_VERIFIED` | Could not confirm a real API call — treat as incomplete |
| `EXPECTED_ERROR` | Self-check probe failed in an expected way |
| `INVALID_INPUT` | Input validation failed before calling RapidX |
| `BUSINESS_ERROR` | RapidX rejected as a business-domain error |
| `NOT_FOUND` | Resource not found after lookup |
| `PERMISSION_SCOPE_ERROR` | Credentials lack required account/portfolio scope |

---

## Preview, Confirmation, and Automation

All write operations require **preview before submission**. Preview and submit must use the same runtime (both CLI or both MCP).

### Manual Confirmation (default)

The agent creates a preview, shows the result to the user, and submits only after the user confirms.

Preview response includes:

```json
{
  "previewId": "rpv_xxx",
  "businessParams": { "symbol": "...", "side": "BUY", ... },
  "riskNotes": [],
  "expiresAt": "2026-06-08T00:05:00.000Z",
  "confirmation": {
    "submitToken": "confirm_rpv_xxx",
    "requiredFields": ["previewId", "continueConsentId"]
  }
}
```

Preview records expire after ~5 minutes. If expired or if submit parameters differ from preview, RapidX returns `BLOCKED` — create a new preview.

### Automation Mode

Automation mode lets an agent submit without a per-order chat confirmation. It is valid **only when the human user explicitly enables it in chat**.

Enable on the **preview request** (not on submit):

```json
{
  "automationMode": true,
  "automationConsentText": "I enable RapidX automation mode for BINANCE_PERP_BTC_USDT with maxNotional 100 and accept automated preview-submit execution.",
  "automationScope": "single-preview"
}
```

Rules:
- Pass the user's exact authorization text through `automationConsentText` — do not rewrite or broaden it.
- `automationConsentText` must include the exact symbol and `maxNotional`.
- The text must clearly express automation intent (`automation`, `automated`, or `自动`).
- Submit still requires `previewId` and `continueConsentId`; automation changes confirmation mode, not the API shape.

When accepted, the preview response includes:

```json
{
  "automation": {
    "enabled": true,
    "confirmationMode": "automation-preview",
    "scope": "single-preview",
    "maxNotional": "100"
  }
}
```

---

## Tool Reference

### Discovery & Diagnostics

| CLI | MCP tool | Description |
|-----|----------|-------------|
| `rapidx schema --json` | `rapidx/tools` | List all capabilities, schemas, and preview requirements |
| `rapidx self-check --read-only --json` | `rapidx/self-check` | Verify read-only connectivity |
| `rapidx update check --json` | `rapidx/update/check` | Check release status and upgrade advice |
| `rapidx doctor --json` | CLI only | Local diagnostics: version, credentials, invocation mode |
| `rapidx auth check` | CLI only | Credential resolution check — no full secrets printed |
| `rapidx invocation check` | CLI only | Verify the agent invocation uses the supported command style |
| `rapidx config list/get/set/unset` | CLI only | Local config helpers (do not replace env vars at runtime) |

### Market Data

Ticker, orderbook, and klines call Binance/OKX public APIs directly. Funding rate, mark price, and symbol info go through RapidX.

| CLI | MCP tool | Data source |
|-----|----------|-------------|
| `rapidx market get-ticker` | `rapidx/market/get-ticker` | Binance / OKX public |
| `rapidx market get-orderbook` | `rapidx/market/get-orderbook` | Binance / OKX public |
| `rapidx market get-klines` | `rapidx/market/get-klines` | Binance / OKX public |
| `rapidx market get-open-interest` | `rapidx/market/get-open-interest` | Binance / OKX public |
| `rapidx market get-funding-rate` | `rapidx/market/get-funding-rate` | RapidX API |
| `rapidx market get-mark-price` | `rapidx/market/get-mark-price` | RapidX API |
| `rapidx market get-symbol-info` | `rapidx/market/get-symbol-info` | RapidX API |

**Key inputs:**

```bash
# symbol required
rapidx market get-ticker        --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-open-interest --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json

# level optional (5/10/20/50/100, default 20)
rapidx market get-orderbook     --input '{"symbol":"BINANCE_PERP_BTC_USDT","level":20}' --json

# interval optional (1m/5m/15m/30m/1h/4h/1d, default 1h); limit default 100
rapidx market get-klines        --input '{"symbol":"BINANCE_PERP_BTC_USDT","interval":"1h","limit":100}' --json

# symbol optional; omit to return all
rapidx market get-funding-rate  --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-mark-price    --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-symbol-info   --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

### Account

| CLI | MCP tool |
|-----|----------|
| `rapidx account overview` | `rapidx/account/overview` |
| `rapidx account balance` | `rapidx/account/balance` |
| `rapidx account set-position-mode` | `rapidx/account/set-position-mode` |

```bash
rapidx account overview --json
rapidx account balance --input '{"mode":"portfolio"}' --json   # portfolio-scoped credentials
rapidx account balance --input '{"mode":"account"}' --json     # account-level credentials required
```

`mode:"account"` with portfolio-scoped credentials returns `PERMISSION_SCOPE_ERROR`.

`set-position-mode` is a write operation — use `rapidx trade preview` with `targetCapabilityId: "account.set-position-mode"` first.

### Orders

All order writes require preview.

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

**Place order:**

```bash
# Preview
rapidx order place-preview --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "orderType": "LIMIT",
  "price": "65000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "agent-001",
  "postOnly": true
}' --json

# Submit
rapidx order place --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "orderType": "LIMIT",
  "price": "65000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "agent-001",
  "postOnly": true,
  "previewId": "rpv_xxx",
  "continueConsentId": "confirm_rpv_xxx"
}' --json
```

Use `quantity` for base/contract quantity. Use `amount` for quote-notional spot market buy. Do not send both.

`orderId` must be a 16-digit RapidX order ID. Wrong format → `RCORE00002`. Valid format but missing → `NOT_FOUND`.

`amend-preview` and `cancel-preview` read back the order first. If the order is already terminal, no usable submit token is returned.

Cancel is asynchronous at the exchange layer. Check `cancelAccepted`, `terminalStateConfirmed`, and `recommendedAction`; poll `rapidx order get` until `CANCELED` if terminal state is not confirmed.

**Read orders:**

```bash
rapidx order get     --input '{"orderId":"1234567890123456"}' --json
rapidx order list    --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx order history --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

### Positions

| CLI | MCP tool |
|-----|----------|
| `rapidx position list` | `rapidx/position/list` |
| `rapidx position history` | `rapidx/position/history` |
| `rapidx position close` | `rapidx/position/close` |
| `rapidx position set-leverage` | `rapidx/position/set-leverage` |

Position writes use `rapidx trade preview`:

```bash
# Set leverage
rapidx trade preview --input '{
  "targetCapabilityId": "position.set-leverage",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "leverage": 5
}' --json

# Close position (no side/quantity — use reduce-only order for partial close)
rapidx trade preview --input '{
  "targetCapabilityId": "position.close",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "reduceOnly": true,
  "maxNotional": "100"
}' --json
```

If no open position exists, `position.close` preview returns `BLOCKED` with `NO_POSITION_TO_CLOSE`.

`maxNotional` is a safety upper bound — not the target close amount.

### Algo Orders

| CLI | MCP tool |
|-----|----------|
| `rapidx algo list` | `rapidx/algo/list` |
| `rapidx algo place` | `rapidx/algo/place` |
| `rapidx algo amend` | `rapidx/algo/amend` |
| `rapidx algo cancel` | `rapidx/algo/cancel` |

Algo writes require preview via `rapidx trade preview` with `targetCapabilityId: "algo.place"` (or `algo.amend`, `algo.cancel`).

TPSL example:

```bash
rapidx trade preview --input '{
  "targetCapabilityId": "algo.place",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "SELL",
  "orderType": "MARKET",
  "maxNotional": "100",
  "clientOrderId": "tpsl-001",
  "algoType": "TPSL",
  "conditionType": "ENTIRE_CLOSE_POSITION",
  "stopLossPrice": "62000",
  "takeProfitPrice": "70000"
}' --json
```

TPSL and entire-close-position algo orders may use `MARKET` execution — this is expected per RapidX rules.

### Live Trade Verification

| CLI | MCP tool |
|-----|----------|
| `rapidx trade verify-live` | `rapidx/trade/verify-live` |

Requires explicit user consent. Submits a real post-only limit order, then queries, amends, cancels, and verifies cleanup.

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

Steps: self-check → symbol rules → internal preview → place → get → amend → cancel → cleanup check.

`rapidx/trading-verification` is kept as a compatibility alias.

---

## Troubleshooting

### Self-check returns NOT_VERIFIED

Check `message`, `code`, and `evidence`. Common causes: missing credentials, wrong `LTP_API_HOST`, missing account permission, blocked network.

| Code | Meaning | Action |
|------|---------|--------|
| `RCORE01001` | Missing credential | Set `LTP_ACCESS_KEY` and `LTP_SECRET_KEY` |
| `RCORE01003` | Missing API host | Set `LTP_API_HOST` |
| `RCORE00002` | Invalid order ID format | Use a 16-digit RapidX `orderId` |
| `RCORE01004` | Credential scope mismatch | Use credentials with the required scope |
| `RCORE22004` | Resource not found | Check `orderId`, `clientOrderId`, or symbol |
| `RCLI20002` | Preview/consent missing | Run the matching preview first |
| `RCLI22001` | RapidX upstream business error | Read the upstream message and adjust |
| `RCORE23002` | Network/timeout failure | Check connectivity and retry |
| `RCORE23003` | Rate limit reached | Retry later; avoid polling loops |

### Write returns BLOCKED

- Was the matching preview called first?
- Is `previewId` present?
- Does `continueConsentId` equal `data.confirmation.submitToken` from the preview?
- Do the submit params match the preview params exactly?
- Is the preview expired (>5 min)?

### MCP tools not visible

Verify MCP config uses `"command": "rapidx", "args": ["mcp", "serve"]`, then reload the agent host.

If the agent host does not inherit shell `PATH`, set `command` to the absolute path from `which rapidx`.

---

## Upgrade

```bash
rapidx update check --json
npm install -g @liquiditytech/rapidx-cli@latest
rapidx schema --json
rapidx self-check --read-only --json
```

Upgrade skills with the same install method used for your agent host.

---

**中文版**: [02-mcp-reference.zh-CN.md](./02-mcp-reference.zh-CN.md)
