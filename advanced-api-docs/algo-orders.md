# Algo Orders

Algo orders are conditional trigger orders — **Take-Profit / Stop-Loss (TPSL)** that auto-execute as `MARKET` close orders when a trigger price is hit. All algo write endpoints (place / replace / cancel) are **asynchronous**: a success response means the request was accepted, not that the order is terminal. Confirm final state with `GET /api/v1/algo/order` or the `ALGO_ORDER` WebSocket push channel.

> **Competition Note**: Only the `TPSL` algo order type is supported in the competition environment. Other algo types (TWAP, VWAP) are rejected by the server even if a preview passes.

**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/algo-orders-rest

**Supported algo order type:**

| `algoOrderType` | Description |
|-----------------|-------------|
| `TPSL` | Take-profit / stop-loss conditional order (TP, SL, or both in one parent order) |

---

## 1. Place Algo Order

**Rate limit**: 3 requests / 50 s

## Endpoint

```
POST /api/v1/algo/order
```

## Parameters

### General

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `clientOrderId` | string | No | Client-defined order ID; lowercase letters (a-z) and digits (0-9) only. Recommended for idempotency |
| `algoOrderType` | string | Yes | `TPSL` (only supported type in the competition) |
| `sym` | string | Yes | LTP symbol, e.g. `BINANCE_PERP_ETH_USDT` |
| `side` | string | No | `BUY` or `SELL` |
| `orderQty` | string | No | Order quantity (OKX = contracts; Binance = coins). Required unless spot market buy |
| `limitPrice` | string | No | Limit price — required for limit orders |
| `positionSide` | string | No | `NONE` (default, one-way) / `LONG` / `SHORT` |
| `reduceOnly` | string | No | `"true"` to allow only position-reducing fills |

### TPSL trigger fields

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `conditionType` | string | Yes (TPSL) | `CONDITIONAL`, `OCO`, `ENTIRE_CLOSE_POSITION`, `PARTIAL_CLOSE_POSITION` |
| `conditionalTriggerPrice` | string | No | Conditional trigger price |
| `conditionalTriggerType` | string | No | `LAST_PRICE` or `MARK_PRICE` |
| `conditionalPrice` | string | No | Fill price after trigger; `0` = market |
| `tpTriggerPrice` | string | No | Take-profit trigger price |
| `tpTriggerType` | string | No | `LAST_PRICE` (default) or `MARK_PRICE` |
| `tpPrice` | string | No | Take-profit fill price; `0` = market |
| `slTriggerPrice` | string | No | Stop-loss trigger price |
| `slTriggerType` | string | No | `LAST_PRICE` (default) or `MARK_PRICE` |
| `slPrice` | string | No | Stop-loss fill price; `0` = market |

## Response (HTTP 200)

```json
{
    "code": 200000,
    "message": "Success",
    "data": {
        "algoOrderId": "2187783984685122",
        "clientOrderId": "eth-tp5pct-20260713-001"
    }
}
```

## Example

TP/SL conditional close (ETH long, +5% TP and −5% SL on the same parent order):

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/algo/order" \
  -X POST \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{
    "algoOrderType": "TPSL",
    "sym": "BINANCE_PERP_ETH_USDT",
    "side": "SELL",
    "orderQty": "0.566",
    "reduceOnly": "true",
    "positionSide": "LONG",
    "conditionType": "CONDITIONAL",
    "conditionalTriggerType": "MARK_PRICE",
    "conditionalTriggerPrice": "1896.73",
    "tpTriggerPrice": "1896.73",
    "tpPrice": "0",
    "slTriggerPrice": "1716.09",
    "slPrice": "0",
    "clientOrderId": "eth-tpsl-20260713-001"
  }' | jq .
```

> Save the returned `algoOrderId` — it is required for replace / cancel / query.
> If both TP and SL are set on one parent order, triggering either side **invalidates the other**. To keep both sides independently active, place two separate parent orders.

---

## 2. Replace Algo Order

**Rate limit**: 3 requests / 50 s

## Endpoint

```
PUT /api/v1/algo/order
```

> Only applies to `TPSL` orders and DMA orders with attached TPSL.

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `algoOrderId` | string | No | Algo order ID (use this or `clientOrderId`) |
| `clientOrderId` | string | No | Client-defined order ID |
| `orderQty` | string | No | New order quantity |
| `conditionalTriggerPrice` | string | No | Conditional trigger price |
| `conditionalTriggerType` | string | No | `LAST_PRICE` or `MARK_PRICE` |
| `conditionalPrice` | string | No | Fill price after trigger; `0` = market |
| `tpTriggerPrice` | string | No | Take-profit trigger price |
| `tpTriggerType` | string | No | `LAST_PRICE` (default) or `MARK_PRICE` |
| `tpPrice` | string | No | Take-profit fill price; `0` = market |
| `slTriggerPrice` | string | No | Stop-loss trigger price |
| `slTriggerType` | string | No | `LAST_PRICE` (default) or `MARK_PRICE` |
| `slPrice` | string | No | Stop-loss fill price; `0` = market |

## Response (HTTP 200)

```json
{
    "algoOrderId": "2187783984685122",
    "clientOrderId": "eth-tpsl-20260713-001"
}
```

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/algo/order" \
  -X PUT \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{
    "algoOrderId": "2187783984685122",
    "tpTriggerPrice": "1987.05",
    "tpPrice": "0",
    "slTriggerPrice": "1716.09",
    "slPrice": "0"
  }' | jq .
```

---

## 3. Cancel Algo Order

**Rate limit**: 3 requests / 50 s

## Endpoint

```
DELETE /api/v1/algo/order
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `algoOrderId` | string | No | Algo order ID |
| `clientOrderId` | string | No | Client-defined order ID |

> Either `algoOrderId` or `clientOrderId` must be sent. Cancel is asynchronous — poll `GET /api/v1/algo/order` until `orderState = CANCELLED`.

## Response (HTTP 200)

```json
{
    "algoOrderId": "2187783984685122",
    "clientOrderId": "eth-tpsl-20260713-001"
}
```

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/algo/order" \
  -X DELETE \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"algoOrderId":"2187783984685122"}' | jq .
```

---

## 4. Cancel All Algo Orders

**Rate limit**: 10 requests / 10 s

## Endpoint

```
DELETE /api/v1/algo/cancelAll
```

## Parameters

All parameters are optional. If omitted, **all** open algo orders under the portfolio are cancelled. Multiple conditions combine with AND logic. `conditionType` only takes effect when `algoOrderType=TPSL`.

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Trading pair, e.g. `BINANCE_PERP_ETH_USDT` |
| `exchangeType` | string | No | `BINANCE` or `OKX` |
| `businessType` | string | No | `PERP` / `SPOT` |
| `algoOrderType` | string | No | `TPSL` (only supported type in the competition) |
| `conditionType` | string | No | `OCO`, `CONDITIONAL`, `ENTIRE_CLOSE_POSITION`, `PARTIAL_CLOSE_POSITION` |

## Response (HTTP 200)

```json
{
    "code": 200000,
    "message": "Success",
    "data": [
        { "algoOrderId": "2127112827963460", "clientOrderId": "2173326802000962" },
        { "algoOrderId": "2127112827963461", "clientOrderId": "2173326802000963" }
    ]
}
```

## Example

Cancel all open TPSL orders on ETH-PERP:

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/algo/cancelAll" \
  -X DELETE \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"sym":"BINANCE_PERP_ETH_USDT","exchangeType":"BINANCE","businessType":"PERP","algoOrderType":"TPSL"}' | jq .
```

---

## 5. Query Algo Order Detail

**Rate limit**: 3 requests / 50 s

## Endpoint

```
GET /api/v1/algo/order
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `algoOrderId` | string | No | Algo order ID |
| `clientOrderId` | string | No | Client-defined order ID |

> Either `algoOrderId` or `clientOrderId` must be sent.

## Response (HTTP 200)

| Field | Type | Description |
|-------|------|-------------|
| `portfolioId` | string | Portfolio ID |
| `algoOrderId` | string | Algo order ID |
| `algoOrderType` | string | `TPSL` |
| `clientOrderId` | string | Client-defined order ID |
| `sym` | string | LTP symbol |
| `exchangeType` | string | `BINANCE` / `OKX` |
| `businessType` | string | `SPOT` / `PERP` |
| `side` | string | `BUY` / `SELL` |
| `limitPrice` | string | Order price |
| `orderQty` | string | Order quantity |
| `positionSide` | string | `NONE` / `LONG` / `SHORT` |
| `reduceOnly` | string | `"true"` or `"false"` |
| `orderState` | string | `PENDING_NEW`, `NEW`, `PROCESSING`, `PENDING_CANCEL`, `CANCELLED`, `COMPLETED`, `REJECTED`, `FAILED`, `EXPIRED` |
| `reason` | string | Failure reason |
| `executedQty` | string | Filled quantity |
| `executedAmount` | string | Filled notional |
| `executedAvgPrice` | string | Average fill price |
| `createAt` | string | Create time |
| `updateAt` | string | Last update time |
| `conditionType` | string | `CONDITIONAL` / `OCO` / `ENTIRE_CLOSE_POSITION` / `PARTIAL_CLOSE_POSITION` |
| `conditionalTriggerPrice` | string | Conditional trigger price |
| `conditionalTriggerPriceType` | string | `LAST_PRICE` / `MARK_PRICE` |
| `conditionalPrice` | string | Fill price after trigger; `0` = market |
| `attachedOrderId` | string | Order ID with attached TP/SL |
| `tpTriggerPrice` | string | Take-profit trigger price |
| `tpTriggerType` | string | `LAST_PRICE` (default) / `MARK_PRICE` |
| `tpPrice` | string | Take-profit fill price |
| `slTriggerPrice` | string | Stop-loss trigger price |
| `slTriggerType` | string | `LAST_PRICE` (default) / `MARK_PRICE` |
| `slPrice` | string | Stop-loss fill price |

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
MSG="&${NONCE}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/algo/order?algoOrderId=2187783984685122" \
  -X GET \
  -H "X-MBX-APIKEY: ${API_KEY}" \
  -H "nonce: ${NONCE}" \
  -H "signature: ${SIG}" | jq .
```

---

## 6. Current Open Algo Orders

**Rate limit**: 3 requests / 50 s

## Endpoint

```
GET /api/v1/algo/openOrders
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Trading pair, e.g. `BINANCE_PERP_ETH_USDT` |
| `exchange` | string | No | `BINANCE` / `OKX` |
| `businessType` | string | No | `SPOT` / `PERP` |
| `begin` | string | No | Begin time |
| `end` | string | No | End time |
| `page` | string | No | Page number (default `1`) |
| `pageSize` | string | No | Page size (default `1000`, max `1000`) |

## Response (HTTP 200)

Array of objects with the same field set as [Query Algo Order Detail](#5-query-algo-order-detail). Only orders in `PENDING_NEW`, `NEW`, `PROCESSING`, or `PENDING_CANCEL` state are returned.

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
MSG="&${NONCE}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/algo/openOrders?sym=BINANCE_PERP_ETH_USDT&exchange=BINANCE&businessType=PERP&page=1&pageSize=1000" \
  -X GET \
  -H "X-MBX-APIKEY: ${API_KEY}" \
  -H "nonce: ${NONCE}" \
  -H "signature: ${SIG}" | jq .
```

---

## Order States

`PENDING_NEW` → `NEW` → `PROCESSING` → `COMPLETED` / `CANCELLED` / `REJECTED` / `FAILED` / `EXPIRED`

## Notes

- All algo write endpoints are **asynchronous**. A success response means the request was accepted — confirm final state by polling `GET /api/v1/algo/order` or subscribing to the `ALGO_ORDER` WebSocket push channel.
- Triggered fills are always `MARKET` orders — slippage may occur in volatile conditions. `conditionalPrice` / `tpPrice` / `slPrice` of `0` requests market execution.
- Trigger prices more than ±15% from the current mark price may be rejected by price-protection rules.
- `clientOrderId` must be globally unique. Reusing a value creates a duplicate order, not a replacement.
- Cancel is irreversible — to re-place after cancelling, run a new place flow and a new `algoOrderId` is generated.
