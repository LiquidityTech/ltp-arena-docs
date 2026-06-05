# RapidX API Reference — Competition Edition

> Complete API surface: CLI commands + MCP tools + REST + WebSocket.

All three transports share the same authentication, error codes, and safety model.

| Transport | When to use |
|-----------|-------------|
| **MCP tools** (`rapidx mcp serve`) | MCP-capable hosts: Claude / Cursor / Codex / Continue |
| **CLI commands** (`rapidx ...`) | CLI-only hosts, shell scripts, CI |
| **REST + WebSocket** | Custom code, low-level integrations |

---

## 1. Authentication

### Required Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `LTP_ACCESS_KEY` | Yes | Your Access Key |
| `LTP_SECRET_KEY` | Yes | Your Secret Key |
| `LTP_API_HOST` | **Yes — no default** | API host provided by the organizer. Missing = `RCORE01003` |

### REST Signature Algorithm

```
1. Sort request params by key (alphabetical):
   sorted_payload = "key1=val1&key2=val2&..."

2. Append "&" + Unix timestamp (seconds):
   payload = sorted_payload + "&" + timestamp
   (no params: payload = "&" + timestamp)

3. HMAC-SHA256 with Secret Key, hex-encoded:
   signature = HMAC-SHA256(secret_key, payload).hexdigest()
```

HTTP headers required:

| Header | Value |
|--------|-------|
| `X-MBX-APIKEY` | Access Key |
| `nonce` | Unix timestamp (string) |
| `signature` | HMAC-SHA256 signature |
| `Content-Type` | `application/json` |

### WebSocket Authentication Signature

Different from REST:

```
message = timestamp + "GET" + "/users/self/verify"
sign = HMAC-SHA256(secret_key, message).hexdigest()
```

---

## 2. Response Envelope

### CLI / MCP Envelope

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
      "timestamp": "2026-06-02T00:00:00.000Z"
    }
  ]
}
```

| Status | Meaning |
|--------|---------|
| `PASS` | Completed successfully |
| `FAIL` | Failed (local, upstream, or unexpected error) |
| `BLOCKED` | Blocked by local validation (e.g. missing preview token) |
| `NOT_VERIFIED` | Could not confirm a real API call |
| `EXPECTED_ERROR` | Self-check probe failed in an expected way |
| `INVALID_INPUT` | Failed local input validation before calling RapidX |
| `BUSINESS_ERROR` | RapidX rejected as a business-domain error |
| `NOT_FOUND` | Requested resource not found after lookup |
| `PERMISSION_SCOPE_ERROR` | Credentials lack required account/portfolio scope |

Error responses may include a `details` object with safe diagnostic fields: `upstreamStatus`, `upstreamCode`, `upstreamMessage`, `hint`, `attempts`.

### REST Envelope

```json
{
  "code": 200000,
  "message": "Success",
  "data": {}
}
```

---

## 3. Preview-Then-Submit Pattern

All write operations require **preview before submission**.

> Preview and submit **must use the same runtime** (both CLI or both MCP — do not mix).

### Order Write Flow

```bash
# Step 1: Preview — returns previewId + submitToken
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
```

Preview response:

```json
{
  "ok": true,
  "status": "PASS",
  "data": {
    "previewId": "rpv_xxx",
    "confirmation": {
      "submitToken": "confirm_rpv_xxx"
    }
  }
}
```

```bash
# Step 2: Submit — same params + previewId + continueConsentId
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

# Step 3: Confirm state
rapidx order list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx position list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

### Non-Order Write Flow (leverage, position mode, position close)

Use `rapidx trade preview` with a `targetCapabilityId`:

```bash
# Set leverage
rapidx trade preview --input '{
  "targetCapabilityId": "position.set-leverage",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "leverage": 5
}' --json

# Set position mode
rapidx trade preview --input '{
  "targetCapabilityId": "account.set-position-mode",
  "exchange": "BINANCE",
  "mode": "NET"
}' --json

# Close position
rapidx trade preview --input '{
  "targetCapabilityId": "position.close",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "reduceOnly": true,
  "maxNotional": "10"
}' --json
```

`position.close` does not accept `side` or `quantity`. For partial close, use a reduce-only order. If no open position exists, preview returns `BLOCKED` with `NO_POSITION_TO_CLOSE`.

`maxNotional` is a safety upper bound — not the target order amount.

### Cancel — Asynchronous at Exchange Layer

A successful cancel response means the cancel was **accepted**, not confirmed. Check `cancelAccepted`, `terminalStateConfirmed`, and `recommendedAction`. If terminal state is not confirmed, poll `rapidx order get` until the order reaches `CANCELED` or another terminal state.

---

## 4. CLI Command Reference

### Format

```bash
rapidx <domain> <action> [--input '<json>' | --input @/absolute/path/file.json] --json
```

### Discovery & Diagnostics

```bash
rapidx schema --json                          # all capabilities + schemas
rapidx self-check --read-only --json          # read-only connectivity check
rapidx doctor --json                          # local diagnostics
rapidx auth check                             # credential check (no secret printed)
rapidx invocation check                       # verify invocation style is supported
rapidx update check --json                    # check for CLI/skills updates
```

### Market Data

```bash
rapidx market get-ticker --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-orderbook --input '{"symbol":"BINANCE_PERP_BTC_USDT","level":20}' --json
rapidx market get-klines --input '{"symbol":"BINANCE_PERP_BTC_USDT","interval":"1h","limit":100}' --json
rapidx market get-funding-rate --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-mark-price --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-symbol-info --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-open-interest --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

### Account

```bash
rapidx account overview --json
rapidx account balance --input '{"mode":"portfolio"}' --json   # portfolio-scoped key
rapidx account balance --input '{"mode":"account"}' --json     # account-level key required
```

`mode:"account"` with a portfolio-scoped key returns `PERMISSION_SCOPE_ERROR`.

### Orders

```bash
# Preview (required before place/amend/cancel)
rapidx order place-preview  --input @order.json --json
rapidx order amend-preview  --input '{"orderId":"1234567890123456","price":"64000"}' --json
rapidx order cancel-preview --input '{"orderId":"1234567890123456"}' --json

# Submit (include previewId + continueConsentId from preview response)
rapidx order place  --input @order_with_consent.json --json
rapidx order amend  --input '{"orderId":"...","price":"64000","previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx"}' --json
rapidx order cancel --input '{"orderId":"...","previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx"}' --json

# Reads
rapidx order get     --input '{"orderId":"1234567890123456"}' --json
rapidx order list    --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx order history --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

`orderId` must be a 16-digit RapidX order ID. Wrong format → `RCORE00002`. Valid format but not found → `NOT_FOUND`.

### Positions

```bash
rapidx position list     --json
rapidx position history  --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json

# Writes — preview first via rapidx trade preview
rapidx position close        --input '{"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx","symbol":"BINANCE_PERP_BTC_USDT","reduceOnly":true,"maxNotional":"10"}' --json
rapidx position set-leverage --input '{"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx","symbol":"BINANCE_PERP_BTC_USDT","leverage":5}' --json
```

### Algo Orders

```bash
# Preview via rapidx trade preview with targetCapabilityId "algo.place"
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

# Submit with previewId + continueConsentId
rapidx algo place  --input '{"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx",...}' --json
rapidx algo amend  --input '{"algoOrderId":"...","previewId":"rpv_xxx","continueConsentId":"..."}' --json
rapidx algo cancel --input '{"algoOrderId":"...","previewId":"rpv_xxx","continueConsentId":"..."}' --json
rapidx algo list   --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

TPSL algo orders may use `MARKET` execution — this is expected per RapidX rules.

### Live Trade Verification

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

Verification steps: self-check → market rule lookup → internal preview → post-only limit submit → order query → amend → cancel → cleanup check.

---

## 5. MCP Tool Reference

### Configuration

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "<your-access-key>",
        "LTP_SECRET_KEY": "<your-secret-key>",
        "LTP_API_HOST": "<provided-api-host>"
      }
    }
  }
}
```

Server info returned on connect:

```json
{
  "protocolVersion": "2025-03-26",
  "serverInfo": { "name": "rapidx", "version": "<current>" },
  "capabilities": { "tools": {} }
}
```

### Tool Names

MCP `tools/call` responses contain a RapidX JSON envelope inside `content[0].text`.

| Category | MCP tool |
|----------|----------|
| Discovery | `rapidx/tools` · `rapidx/self-check` · `rapidx/update/check` |
| Market | `rapidx/market/get-ticker` · `rapidx/market/get-orderbook` · `rapidx/market/get-klines` · `rapidx/market/get-funding-rate` · `rapidx/market/get-mark-price` · `rapidx/market/get-symbol-info` · `rapidx/market/get-open-interest` |
| Account | `rapidx/account/overview` · `rapidx/account/balance` · `rapidx/account/set-position-mode` |
| Order | `rapidx/order/place-preview` · `rapidx/order/place` · `rapidx/order/amend-preview` · `rapidx/order/amend` · `rapidx/order/cancel-preview` · `rapidx/order/cancel` · `rapidx/order/get` · `rapidx/order/list` · `rapidx/order/history` |
| Trade utils | `rapidx/trade/preview` · `rapidx/trade/verify-live` · `rapidx/order/preview` (alias) |
| Position | `rapidx/position/list` · `rapidx/position/history` · `rapidx/position/close` · `rapidx/position/set-leverage` |
| Algo | `rapidx/algo/list` · `rapidx/algo/place` · `rapidx/algo/amend` · `rapidx/algo/cancel` |

`rapidx/trading-verification` is kept as a compatibility alias for `rapidx/trade/verify-live`.

---

## 6. Key Field Reference

### Order Place / Preview

| Field | Required | Description |
|-------|----------|-------------|
| `symbol` | Yes | LTP symbol, e.g. `BINANCE_PERP_BTC_USDT` |
| `side` | Yes | `BUY` / `SELL` |
| `orderType` | Yes | `LIMIT` / `MARKET` |
| `price` | For LIMIT | Limit price |
| `quantity` | Conditional | Base/contract quantity. Use `amount` for quote-notional spot market buy — do not send both |
| `maxNotional` | Yes | Safety upper bound in quote currency |
| `clientOrderId` | Recommended | Idempotency key, used for state confirmation and troubleshooting |
| `postOnly` | No | `true` forces GTX (post-only); order rejected if it would immediately fill |
| `previewId` | Submit only | From preview response `data.previewId` |
| `continueConsentId` | Submit only | From preview response `data.confirmation.submitToken` |

> Check symbol rules via `rapidx market get-symbol-info` before sizing: `minNotional`, `lotSize`, `tickSize`, `contractSize` all vary.

### Order State Values

| `orderState` | Description |
|-------------|-------------|
| `NEW` | Acknowledged, awaiting exchange confirmation |
| `OPEN` | Resting on the order book |
| `PARTIALLY_FILLED` | Partially executed |
| `FILLED` | Fully executed |
| `CANCELED` | Cancelled |
| `REJECTED` | Rejected by exchange |

### Account Balance

```bash
# Portfolio-scoped credentials (competition default)
rapidx account balance --input '{"mode":"portfolio"}' --json

# Account-level credentials required for:
rapidx account balance --input '{"mode":"account"}' --json
```

---

## 7. Raw REST API Endpoints

| Service | Method | Path |
|---------|--------|------|
| Account | GET | `/api/v1/trading/account` |
| Assets | GET | `/api/v1/trading/portfolio/assets` |
| Place Order | POST | `/api/v1/trading/order` |
| Amend Order | PUT | `/api/v1/trading/order` |
| Cancel Order | DELETE | `/api/v1/trading/order` |
| Cancel All | DELETE | `/api/v1/trading/cancelAll` |
| Get Order | GET | `/api/v1/trading/order` |
| Open Orders | GET | `/api/v1/trading/orders` |
| Order History | GET | `/api/v1/trading/history/orders` |
| Positions | GET | `/api/v1/trading/position` |
| Position History | GET | `/api/v1/trading/history/position` |
| Close Position | DELETE | `/api/v1/trading/position` |
| Get Leverage | GET | `/api/v1/trading/perp/leverage` |
| Set Leverage | POST | `/api/v1/trading/position/leverage` |
| Executions | GET | `/api/v1/trading/executions` |
| Statement | GET | `/api/v1/trading/statement` |
| Symbol Info | GET | `/api/v1/trading/sym/info` |
| Funding Rate | GET | `/api/v1/market/fundingRate` |
| Mark Price | GET | `/api/v1/market/markPrice` |
| User Fee Rate | GET | `/api/v1/trading/userFeeRate` |

Full REST details: [`api-reference.md`](api-reference.md).

---

## 8. WebSocket

### Private WebSocket

**URL**: `wss://wss-uat.liquiditytech.com/v1/private`

After login, the `Orders` channel auto-pushes order state changes:

```json
{
  "channel": "Orders",
  "data": {
    "orderId": "1000000000000003",
    "orderState": "FILLED",
    "executedQty": "0.001",
    "executedAvgPrice": "65000"
  }
}
```

Actions via WebSocket:

```json
{ "id": "p1", "action": "place_order",  "args": { ... } }
{ "id": "r1", "action": "replace_order","args": { ... } }
{ "id": "c1", "action": "cancel_order", "args": { ... } }
```

### Market Data WebSocket

**URL**: `wss://mds-uat.liquiditytech.com/marketdata/v2/public`

Append `?binary=false` for plain-text JSON.

| Channel | Frequency | Scope |
|---------|-----------|-------|
| `BBO` | On change | All |
| `TICKER` | 2000 ms | All |
| `TRADE` | Real-time | All |
| `ORDER_BOOK` | 250 ms | All |
| `KLINE` | Real-time | All |
| `MARK_PRICE` | Real-time | Perpetual only |
| `INDEX_PRICE` | Real-time | Perpetual only |
| `OPEN_INTEREST` | On change | Perpetual only |

Subscribe:

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO",   "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TRADE", "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

Limits by **trading pair** (not channel): BBO + TICKER + TRADE on same symbol = 1 pair.

---

## 9. Error Codes

### CLI / MCP Error Code Format

| Prefix | Scope |
|--------|-------|
| `RCORE*` | Shared by CLI and MCP |
| `RCLI*` | CLI-specific |
| `RMCP*` | MCP-specific |

| Code | Meaning | Action |
|------|---------|--------|
| `RCORE01001` | Missing credential | Configure `LTP_ACCESS_KEY` and `LTP_SECRET_KEY` |
| `RCORE01003` | Missing API host | Configure `LTP_API_HOST` |
| `RCORE00002` | Invalid order ID format | Use a 16-digit RapidX `orderId` |
| `RCORE01004` | Credential scope mismatch | Use credentials with the required scope |
| `RCORE22004` | Resource not found | Check `orderId`, `clientOrderId`, or symbol |
| `RCLI20002` | Preview/consent token missing | Run the matching preview first |
| `RCLI22001` | RapidX upstream business error | Read the upstream message and adjust the request |
| `RCORE23002` | Network or timeout failure | Check connectivity and retry |
| `RCORE23003` | Rate limit reached | Retry later; avoid polling loops |

### REST Error Codes

| Code | Meaning |
|------|---------|
| `200000` | Success |
| `2002` | Invalid API authorization |
| `401018` | Order not found |
| `401097` | Position quantity is 0, no need to close |
| `401117` | Order already completed |

---

## 10. Troubleshooting

### npm cannot find the package

```bash
npm config get @liquiditytech:registry
# Expected: https://registry.npmjs.org/

npm config set @liquiditytech:registry https://registry.npmjs.org/
npm install -g @liquiditytech/rapidx-cli@latest
```

### Write operation returns BLOCKED

- Matching preview was called first? `previewId` is present?
- `continueConsentId` equals `data.confirmation.submitToken` from the preview response?
- Submit uses the same business parameters as the preview?
- Check `rapidx schema --json` for current field requirements.

### MCP tools not visible

```json
{ "command": "rapidx", "args": ["mcp", "serve"] }
```

Reload agent host after editing MCP config. Inspect `rapidx/tools` in the tool list.

### After upgrade, old version still shows

```bash
which rapidx
rapidx --version
```

If MCP host does not inherit shell PATH, set `command` to the absolute path from `which rapidx`.

### Upgrade

```bash
rapidx update check --json
npm install -g @liquiditytech/rapidx-cli@latest
rapidx schema --json
rapidx self-check --read-only --json
```

---

**中文版**: [rapidx-api.zh-CN.md](./rapidx-api.zh-CN.md)
