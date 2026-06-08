# Advanced: Direct REST & WebSocket API

> For custom code and low-level integrations that call RapidX directly, without the CLI or MCP layer.

**Most participants should use the CLI or MCP** — see [`01-quickstart.md`](./01-quickstart.md). This document is for cases where you need direct HTTP or WebSocket access: performance-critical bots, languages without Node.js, or custom tooling.

---

## Authentication

### Required Headers (REST)

| Header | Value |
|--------|-------|
| `X-MBX-APIKEY` | Your Access Key |
| `nonce` | Current Unix timestamp in seconds (string) |
| `signature` | HMAC-SHA256 signature (see below) |
| `Content-Type` | `application/json` |

### REST Signature Algorithm

```
1. Sort request parameters alphabetically by key:
   sorted_payload = "key1=val1&key2=val2&..."

2. Append "&" + timestamp:
   payload = sorted_payload + "&" + timestamp
   (no parameters: payload = "&" + timestamp)

3. HMAC-SHA256 with your Secret Key, hex-encoded:
   signature = HMAC-SHA256(secret_key, payload).hexdigest()
```

**Python example:**

```python
import hmac, hashlib, time

def sign(params: dict, secret_key: str):
    timestamp = str(int(time.time()))
    sorted_payload = "&".join(f"{k}={v}" for k, v in sorted(params.items()))
    payload = (sorted_payload + "&" if sorted_payload else "&") + timestamp
    sig = hmac.new(secret_key.encode(), payload.encode(), hashlib.sha256).hexdigest()
    return sig, timestamp
```

### WebSocket Authentication Signature

Different from REST — used for the private WebSocket login:

```
message = timestamp + "GET" + "/users/self/verify"
sign = HMAC-SHA256(secret_key, message).hexdigest()
```

---

## REST API Endpoints

Base URL: the `LTP_API_HOST` value provided by the organizer.

### Account & Assets

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/trading/account` | Account overview per exchange |
| GET | `/api/v1/trading/portfolio/assets` | Portfolio asset breakdown |
| GET | `/api/v1/trading/user/tradingStats` | Trading statistics (`begin`, `end` params) |
| GET | `/api/v1/trading/userFeeRate` | Maker/Taker fee rates |

### Orders

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/trading/order` | Place order |
| PUT | `/api/v1/trading/order` | Amend order |
| DELETE | `/api/v1/trading/order` | Cancel order |
| DELETE | `/api/v1/trading/cancelAll` | Cancel all orders (`sym` or `exchangeType`) |
| GET | `/api/v1/trading/order` | Get single order (`orderId` or `clientOrderId`) |
| GET | `/api/v1/trading/orders` | List open orders |
| GET | `/api/v1/trading/history/orders` | Order history (`sym` optional) |
| GET | `/api/v1/trading/archive/history/orders` | Archived order history |
| GET | `/api/v1/trading/executions` | Execution records |
| GET | `/api/v1/trading/executions/pageable` | Pageable executions |
| GET | `/api/v1/trading/statement` | Trading statement |

**Place order fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `sym` | Yes | e.g. `BINANCE_PERP_BTC_USDT` |
| `side` | Yes | `BUY` / `SELL` |
| `positionSide` | Yes (hedge mode) | `LONG` / `SHORT` |
| `orderType` | Yes | `LIMIT` / `MARKET` |
| `orderQty` | Yes | Base/contract quantity |
| `limitPrice` | For LIMIT | Limit price |
| `timeInForce` | No | `GTC` (default) / `IOC` / `FOK` / `GTX` |
| `clientOrderId` | Recommended | Max 40 chars |

**Order states:** `NEW` → `OPEN` → `PARTIALLY_FILLED` → `FILLED` / `CANCELLED` / `REJECTED`

### Positions

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/trading/position` | Open positions (`sym`, `exchange` optional) |
| GET | `/api/v1/trading/history/position` | Position history |
| DELETE | `/api/v1/trading/position` | Close position (`sym`, `positionSide`) |
| DELETE | `/api/v1/trading/positions` | Close all positions (`exchangeType`, `closeAllPos:"true"`) |
| GET | `/api/v1/trading/perp/leverage` | Get leverage (`sym` or `exchange` optional) |
| POST | `/api/v1/trading/position/leverage` | Set leverage (`sym`, `leverage`) |
| GET | `/api/v1/adl/rank` | ADL rank (`sym` optional) |

### Market Data (RapidX)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/trading/sym/info` | Symbol rules (`sym` optional) |
| GET | `/api/v1/market/fundingRate` | Funding rate (`sym` required) |
| GET | `/api/v1/market/markPrice` | Mark price (`sym` optional) |
| GET | `/api/v1/trading/positionBracket` | Position tier/bracket |
| GET | `/api/v1/trading/loan/info` | Loan info |
| GET | `/api/v1/trading/coin/discount` | Coin discount rate |
| GET | `/api/v1/trading/margin/leverage` | Margin leverage |

### REST Response Format

```json
{
  "code": 200000,
  "message": "Success",
  "data": { ... }
}
```

`code: 200000` = success. Any other value is an error.

---

## WebSocket

### Private WebSocket (Trading + Account Events)

**URL:** `wss://wss-uat.liquiditytech.com/v1/private`

**Login:**

```json
{
  "action": "login",
  "args": {
    "apiKey": "YOUR_ACCESS_KEY",
    "timestamp": "1778140847",
    "sign": "..."
  }
}
```

**Response:**

```json
{ "event": "login", "code": 0, "msg": "" }
```

**Order actions:**

```json
{ "id": "p1", "action": "place_order",  "args": { ... } }
{ "id": "a1", "action": "replace_order","args": { "orderId": "...", "replacePrice": "64000" } }
{ "id": "c1", "action": "cancel_order", "args": { "orderId": "..." } }
```

**Orders push channel** (auto-subscribed after login):

```json
{
  "channel": "Orders",
  "data": {
    "orderId": "1234567890123456",
    "clientOrderId": "agent-001",
    "orderState": "FILLED",
    "executedQty": "0.001",
    "executedAvgPrice": "65000"
  }
}
```

Keepalive: send `"ping"` every 15 seconds; server replies `"pong"`.

### Market Data WebSocket

**URL:** `wss://mds-uat.liquiditytech.com/marketdata/v2/public`

Append `?binary=false` for plain-text JSON instead of GZIP frames.

**Rate limits:**

| Mode | Max connections / IP | Max symbols / connection |
|------|---------------------|--------------------------|
| Unauthenticated | 5 | 5 |
| Authenticated | 40 | 50 |

Limits are counted by **trading pair** — subscribing BBO + TICKER + TRADE for the same symbol counts as 1 pair.

**Subscribe:**

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO",   "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TRADE", "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

**Available channels:**

| Channel | Frequency | Scope |
|---------|-----------|-------|
| `BBO` | On change | All |
| `TICKER` | 2000 ms | All |
| `TRADE` | Real-time | All |
| `ORDER_BOOK` | 250 ms | All |
| `KLINE` | Real-time | All |
| `MARK_PRICE` | Real-time | Perpetual only |
| `INDEX_PRICE` | Real-time | Perpetual only |
| `MARK_PRICE_KLINE` | Real-time | Perpetual only |
| `INDEX_KLINE` | Real-time | Perpetual only |
| `OPEN_INTEREST` | On change | Perpetual only |

Keepalive: send `{ "ping": <timestamp_ms> }` every 20 seconds; server replies `{ "pong": <timestamp_ms> }`.

---

## Symbol Format

```
{EXCHANGE}_{TYPE}_{BASE}_{QUOTE}
```

| Exchange | Type | Examples |
|----------|------|---------|
| `BINANCE` | `PERP` | `BINANCE_PERP_BTC_USDT`, `BINANCE_PERP_ETH_USDT` |
| `OKX` | `PERP` | `OKX_PERP_BTC_USDT`, `OKX_PERP_ETH_USDT` |
| `EDX` | `PERP` | `EDX_PERP_ETH_USDT` |

Channels marked "Perpetual only" reject `SPOT` symbols with error `11100`.

---

## Common Error Codes

### REST

| Code | Meaning |
|------|---------|
| `200000` | Success |
| `2002` | Invalid API authorization |
| `401018` | Order not found |
| `401097` | Position quantity is 0 |
| `401117` | Order already completed |

### Market Data WebSocket

| Code | Meaning |
|------|---------|
| `0` | Success |
| `11100` | Invalid symbol |
| `11210` | Unauthenticated symbol limit (5) exceeded |
| `11220` | Authenticated symbol limit (50) exceeded |
| `11250` | Symbol not currently supported |
| `11260` | Request rate limit exceeded |

---

**中文版**: [03-advanced-api.zh-CN.md](./03-advanced-api.zh-CN.md)
