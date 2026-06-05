# LTP RapidX API Reference

> **AI Quantitative Trading Competition — UAT Simulation Environment**

---

## Table of Contents

1. [Endpoints](#1-endpoints)
2. [Authentication](#2-authentication)
3. [Response Format](#3-response-format)
4. [REST API](#4-rest-api)
   - 4.1 Account
   - 4.2 Assets
   - 4.3 Orders
   - 4.4 Positions
   - 4.5 Executions
   - 4.6 Statements
   - 4.7 Market & Rules
5. [WebSocket API](#5-websocket-api)
   - 5.1 Private WebSocket (Trading)
   - 5.2 Market Data WebSocket
6. [Symbol Format](#6-symbol-format)
7. [Important Notes](#7-important-notes)

---

## 1. Endpoints

| Type | URL |
|------|-----|
| REST API | `https://uat-api.liquiditytech.com` |
| Private WebSocket | `wss://wss-uat.liquiditytech.com/v1/private` |
| Market Data WebSocket | `wss://mds-uat.liquiditytech.com/marketdata/v2/public` |

> **Important**: Orders placed in the UAT environment are routed to real exchange accounts. Use only the API credentials provided for the competition.

### Market Data WebSocket — Rate Limits

| Mode | Max Connections | Max Symbols |
|------|----------------|-------------|
| Unauthenticated | 5 | 5 |
| Authenticated (with API Key) | 40 | 50 |

> Quantitative strategies typically require multiple symbols. **Authenticating the market data WebSocket is strongly recommended** to obtain the higher limits. The login procedure is the same as for the private WebSocket (see §5.1).

---

## 2. Authentication

All REST requests must include the following HTTP headers:

| Header | Description |
|--------|-------------|
| `X-MBX-APIKEY` | Your API Key |
| `nonce` | Current Unix timestamp in seconds (string) |
| `signature` | HMAC-SHA256 signature (see below) |
| `Content-Type` | `application/json` |

### Signature Algorithm

```
1. Sort all request parameters alphabetically by key and join them:
   sorted_payload = "key1=val1&key2=val2&..."

2. Append & and the timestamp:
   payload = sorted_payload + "&" + timestamp
   (if no parameters: payload = "&" + timestamp)

3. Compute HMAC-SHA256 using your Secret Key, hex-encoded:
   signature = HMAC-SHA256(secret_key, payload).hexdigest()
```

**Python example:**

```python
import hmac, hashlib, time

def sign(params: dict, secret_key: str) -> tuple[str, str]:
    timestamp = str(int(time.time()))
    sorted_payload = "&".join(f"{k}={v}" for k, v in sorted(params.items()))
    payload = (sorted_payload + "&" if sorted_payload else "&") + timestamp
    signature = hmac.new(secret_key.encode(), payload.encode(), hashlib.sha256).hexdigest()
    return signature, timestamp
```

---

## 3. Response Format

```json
{
  "code": 200000,
  "message": "Success",
  "data": { ... }
}
```

| Field | Description |
|-------|-------------|
| `code` | `200000` = success; any other value is an error code |
| `message` | Status description |
| `data` | Business payload; structure varies per endpoint |

---

## 4. REST API

### 4.1 Account

#### Query Account

```
GET /api/v1/trading/account
```

No parameters. Returns margin, risk, and position summary for each sub-account.

**Response `data` (array, one element per exchange account):**

```json
[
  {
    "portfolioId": "1000000000000001",
    "exchangeType": "BINANCE",
    "equity": "102.23",
    "availableMargin": "93.79",
    "maintainMargin": "0.29",
    "riskRatio": "0.0029",
    "accountStatus": "NORMAL",
    "positionMode": "BOTH",
    "upnl": "0"
  }
]
```

| Field | Description |
|-------|-------------|
| `exchangeType` | Exchange: `BINANCE` / `OKX` / `EDX` |
| `equity` | Account equity (USDT) |
| `availableMargin` | Available margin (USDT) |
| `riskRatio` | Risk ratio |
| `accountStatus` | `NORMAL` / `MARGIN_CALL` / `LIQUIDATION` |
| `positionMode` | `BOTH` (hedge mode) / `NET` (one-way mode) |

---

### 4.2 Assets

#### Query Portfolio Assets

```
GET /api/v1/trading/portfolio/assets
```

No parameters. Returns coin-level asset breakdown per exchange.

**Response `data.list` (array):**

```json
{
  "page": 1,
  "pageSize": 1000,
  "totalSize": 5,
  "list": [
    {
      "portfolioId": "1000000000000001",
      "coin": "USDT",
      "exchangeType": "BINANCE",
      "balance": "107.98",
      "available": "101.98",
      "frozen": "6",
      "equity": "107.98",
      "maxTransferable": "96.19"
    }
  ]
}
```

---

### 4.3 Orders

#### Place Order

```
POST /api/v1/trading/order
```

**Request body (JSON):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sym` | string | ✓ | Symbol, e.g. `BINANCE_PERP_BTC_USDT` |
| `side` | string | ✓ | `BUY` / `SELL` |
| `positionSide` | string | ✓ | `LONG` / `SHORT` (required in hedge mode) |
| `orderType` | string | ✓ | `LIMIT` / `MARKET` |
| `timeInForce` | string | for LIMIT | `GTC` / `IOC` / `FOK` |
| `orderQty` | string | ✓ | Order quantity in base currency (e.g. BTC) |
| `limitPrice` | string | for LIMIT | Limit price |
| `clientOrderId` | string | recommended | Client-assigned order ID, max 40 chars |

**Response `data`:**

```json
{
  "orderId": "1000000000000002",
  "clientOrderId": "my_order_001"
}
```

---

#### Amend Order

```
PUT /api/v1/trading/order
```

| Field | Type | Description |
|-------|------|-------------|
| `orderId` | string | System order ID (or `clientOrderId`) |
| `clientOrderId` | string | Client order ID (or `orderId`) |
| `replacePrice` | string | New price (optional) |
| `replaceQty` | string | New quantity (optional) |

**Response `data`:**

```json
{
  "orderId": "1000000000000002",
  "orderState": "OPEN"
}
```

---

#### Cancel Order

```
DELETE /api/v1/trading/order
```

| Field | Type | Description |
|-------|------|-------------|
| `orderId` | string | System order ID (or `clientOrderId`) |
| `clientOrderId` | string | Client order ID (or `orderId`) |

**Response `data`:**

```json
{
  "orderId": "1000000000000002",
  "clientOrderId": "my_order_001",
  "orderState": "OPEN",
  "action": "CANCEL_PENDING"
}
```

---

#### Cancel All Orders

```
DELETE /api/v1/trading/cancelAll
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Cancel all orders for a specific symbol |
| `exchangeType` | string | Cancel all orders on a specific exchange |

---

#### Query Order

```
GET /api/v1/trading/order
```

| Field | Type | Description |
|-------|------|-------------|
| `orderId` | string | System order ID (or `clientOrderId`) |
| `clientOrderId` | string | Client order ID (or `orderId`) |

**Response `data` fields:**

| Field | Description |
|-------|-------------|
| `orderState` | `NEW` / `OPEN` / `PARTIALLY_FILLED` / `FILLED` / `CANCELLED` / `REJECTED` |
| `executedQty` | Filled quantity |
| `executedAvgPrice` | Average fill price |
| `limitPrice` | Order price |
| `orderQty` | Order quantity |
| `fee` | Trading fee |
| `leverage` | Leverage at time of order |

---

#### Query Open Orders

```
GET /api/v1/trading/orders
```

No parameters. Returns all active orders in `data.list`.

---

#### Query Order History

```
GET /api/v1/trading/history/orders
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

#### Query Order History (Archive)

```
GET /api/v1/trading/archive/history/orders
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

### 4.4 Positions

#### Query Positions

```
GET /api/v1/trading/position
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |
| `exchange` | string | Optional; filter by exchange |

---

#### Query Position History

```
GET /api/v1/trading/history/position
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

#### Query ADL Rank

```
GET /api/v1/adl/rank
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

#### Get Leverage

```
GET /api/v1/trading/perp/leverage
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |
| `exchange` | string | Optional; filter by exchange |

**Response `data` (array):**

```json
[
  { "sym": "BINANCE_PERP_BTC_USDT", "leverage": "5" }
]
```

---

#### Set Leverage

```
POST /api/v1/trading/position/leverage
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sym` | string | ✓ | Symbol |
| `leverage` | string | ✓ | Leverage multiplier, e.g. `"5"` |

> Ensure `availableMargin × leverage ≥ order notional × 1.1` before placing an order. Call this endpoint to adjust leverage as needed.

---

#### Close Position

```
DELETE /api/v1/trading/position
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sym` | string | ✓ | Symbol |
| `positionSide` | string | ✓ | `LONG` / `SHORT` |

---

#### Close All Positions

```
DELETE /api/v1/trading/positions
```

| Field | Type | Description |
|-------|------|-------------|
| `exchangeType` | string | Target exchange |
| `closeAllPos` | string | `"true"` to close all positions |

---

### 4.5 Executions

#### Query Executions

```
GET /api/v1/trading/executions
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

#### Query Executions (Pageable)

```
GET /api/v1/trading/executions/pageable
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

#### Query Executions (Archive)

```
GET /api/v1/trading/archive/executions/pageable
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

### 4.6 Statements

#### Query Trading Statement

```
GET /api/v1/trading/statement
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; filter by symbol |

---

#### Query Trading Statistics

```
GET /api/v1/trading/user/tradingStats
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `begin` | string | ✓ | Start timestamp (seconds) |
| `end` | string | ✓ | End timestamp (seconds) |

---

### 4.7 Market & Rules

#### Query Symbol Info

```
GET /api/v1/trading/sym/info
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; omit to return all symbols |

Returns minimum order quantity, price precision, quantity precision, maximum leverage, etc.

---

#### Query Funding Rate

```
GET /api/v1/market/fundingRate
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sym` | string | ✓ | Symbol |

---

#### Query Mark Price

```
GET /api/v1/market/markPrice
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; omit to return all symbols |

---

#### Query User Fee Rate

```
GET /api/v1/trading/userFeeRate
```

No parameters. Returns Maker/Taker fee rates for the account.

---

#### Query Position Bracket (Tier)

```
GET /api/v1/trading/positionBracket
```

| Field | Type | Description |
|-------|------|-------------|
| `sym` | string | Optional; omit to return all symbols |

---

#### Query Loan Info

```
GET /api/v1/trading/loan/info
```

| Field | Type | Description |
|-------|------|-------------|
| `exchangeType` | string | Optional, e.g. `BINANCE` |
| `coin` | string | Optional, e.g. `BTC` |

---

#### Query Coin Discount Rate

```
GET /api/v1/trading/coin/discount
```

| Field | Type | Description |
|-------|------|-------------|
| `coin` | string | Optional, e.g. `BTC` |

---

#### Query Margin Leverage

```
GET /api/v1/trading/margin/leverage
```

| Field | Type | Description |
|-------|------|-------------|
| `exchangeType` | string | Optional, e.g. `BINANCE` |
| `coin` | string | Optional, e.g. `BTC` |

---

## 5. WebSocket API

### 5.1 Private WebSocket (Trading)

**URL:** `wss://wss-uat.liquiditytech.com/v1/private`

#### Login

Authentication is required after connecting. Send a login message before any other action.

**Signature:** `HMAC-SHA256(secret_key, timestamp + "GET" + "/users/self/verify")`, hex-encoded.

**Send:**

```json
{
  "action": "login",
  "args": {
    "apiKey": "YOUR_API_KEY",
    "timestamp": "1778140847",
    "sign": "e54702b1..."
  }
}
```

**Response (success):**

```json
{
  "event": "login",
  "code": 0,
  "msg": ""
}
```

---

#### Place Order

```json
{
  "id": "place-1",
  "action": "place_order",
  "args": {
    "clientOrderId": "my_order_001",
    "sym": "BINANCE_PERP_BTC_USDT",
    "side": "BUY",
    "positionSide": "LONG",
    "orderType": "LIMIT",
    "timeInForce": "GTC",
    "orderQty": "0.003",
    "limitPrice": "80000"
  }
}
```

**Response:**

```json
{
  "id": "place-1",
  "event": "place_order",
  "code": 200000,
  "msg": "Success",
  "data": {
    "orderId": "1000000000000003",
    "clientOrderId": "my_order_001"
  }
}
```

---

#### Amend Order

```json
{
  "id": "replace-1",
  "action": "replace_order",
  "args": {
    "orderId": "1000000000000003",
    "replacePrice": "79000"
  }
}
```

**Response:**

```json
{
  "id": "replace-1",
  "event": "replace_order",
  "code": 200000,
  "data": {
    "orderId": "1000000000000003",
    "orderState": "OPEN"
  }
}
```

---

#### Cancel Order

```json
{
  "id": "cancel-1",
  "action": "cancel_order",
  "args": {
    "orderId": "1000000000000003"
  }
}
```

---

#### Order Push — `Orders` Channel

Order state changes are pushed automatically after login. No subscription is required.

```json
{
  "channel": "Orders",
  "data": {
    "orderId": "1000000000000003",
    "clientOrderId": "my_order_001",
    "orderState": "FILLED",
    "action": "AMEND_COMPLETED",
    "executedQty": "0.003",
    "executedAvgPrice": "80000"
  }
}
```

| `orderState` | Description |
|--------------|-------------|
| `NEW` | Acknowledged, awaiting exchange confirmation |
| `OPEN` | Resting on the order book |
| `PARTIALLY_FILLED` | Partially executed |
| `FILLED` | Fully executed |
| `CANCELLED` | Cancelled |
| `REJECTED` | Rejected by exchange |

#### Keepalive

Send `"ping"` every 15 seconds to keep the connection alive. The server responds with `"pong"`.

---

### 5.2 Market Data WebSocket

**URL:** `wss://mds-uat.liquiditytech.com/marketdata/v2/public`

Append `?binary=false` to receive plain-text JSON instead of GZIP-compressed frames:

```
wss://mds-uat.liquiditytech.com/marketdata/v2/public?binary=false
```

Authentication is optional but strongly recommended (see rate limits in §1).

#### Subscribe

Multiple channels can be combined in a single request.

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO",    "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TICKER", "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TRADE",  "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

#### Available Channels

| Channel | Description | Frequency | Scope |
|---------|-------------|-----------|-------|
| `BBO` | Best bid & offer | On change | All |
| `TICKER` | 24-hour rolling ticker | 2000 ms | All |
| `TRADE` | Latest trades | Real-time | All |
| `ORDER_BOOK` | Order book depth | 250 ms | All |
| `KLINE` | Candlestick (trade price) | Real-time | All |
| `MARK_PRICE` | Mark price | Real-time | Futures only |
| `INDEX_PRICE` | Index price | Real-time | Futures only |
| `MARK_PRICE_KLINE` | Candlestick (mark price) | Real-time | Futures only |
| `INDEX_KLINE` | Candlestick (index price) | Real-time | Futures only |
| `OPEN_INTEREST` | Open interest | On change | Futures only |

> Limits are counted by **trading pair**, not by channel. Subscribing to `BBO`, `TICKER`, and `TRADE` for the same symbol counts as 1 pair.

#### BBO Push Example

```json
{
  "arg": { "channel": "BBO", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": {
    "bid": "81422.3",
    "bidqty": "0.5",
    "ask": "81422.4",
    "askqty": "0.3"
  }
}
```

#### Keepalive

Send `{ "ping": <timestamp_ms> }` every 20 seconds. The server responds with `{ "pong": <timestamp_ms> }`.

---

## 6. Symbol Format

Pattern: `{EXCHANGE}_{TYPE}_{BASE}_{QUOTE}`

| Symbol | Description |
|--------|-------------|
| `BINANCE_PERP_BTC_USDT` | Binance BTC perpetual futures |
| `BINANCE_PERP_ETH_USDT` | Binance ETH perpetual futures |
| `BINANCE_PERP_XRP_USDT` | Binance XRP perpetual futures |
| `BINANCE_PERP_BNB_USDT` | Binance BNB perpetual futures |
| `BINANCE_PERP_ADA_USDT` | Binance ADA perpetual futures |
| `OKX_PERP_BTC_USDT` | OKX BTC perpetual futures |
| `OKX_PERP_ETH_USDT` | OKX ETH perpetual futures |
| `OKX_PERP_ADA_USDT` | OKX ADA perpetual futures |
| `EDX_PERP_ETH_USDT` | EDX ETH perpetual futures |

Use `GET /api/v1/trading/sym/info` to retrieve the full list of tradable symbols.

---

## 7. Important Notes

### Pre-Order Checklist

1. **Leverage**: Call `POST /api/v1/trading/position/leverage` before placing an order. Ensure `availableMargin × leverage ≥ order notional × 1.1` (10% safety buffer).
2. **Hedge mode (BOTH)**: Accounts default to hedge position mode. The `positionSide` field (`LONG` or `SHORT`) is **mandatory** in all order requests — omitting it will cause the order to be rejected.
3. **Minimum order size**: Varies by symbol. Query the `minQty` field via `GET /api/v1/trading/sym/info`.

### Order Lifecycle

- After cancellation, the order briefly remains in state `OPEN` with `action: CANCEL_PENDING` before transitioning to `CANCELLED`.
- WebSocket push notifications may arrive with some delay. Use the REST `GET /api/v1/trading/order` endpoint as a fallback to confirm order state when a push does not arrive within the expected window.
