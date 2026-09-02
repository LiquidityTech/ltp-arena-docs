# Advanced: Direct REST & WebSocket API

> For custom code and low-level integrations that call RapidX directly, without the CLI or MCP layer.

**Most participants should use the CLI or MCP** — see [`01-quickstart.md`](./01-quickstart.md). This document is for cases where you need direct HTTP or WebSocket access: performance-critical bots, languages without Node.js, or custom tooling.


Official API docs: [get-account-list](../advanced-api-docs/get-account-list.md)

---

## Authentication

[Official docs →](../advanced-api-docs/authentication.md)

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

Base URL: Phase I (Sandbox) `https://api.ltp-contest.com` · Phase II (Mainnet) `https://api.liquiditytech.com`

> For leaderboard/ranking data, see [Leaderboard API](../advanced-api-docs/leaderboard.md).

### Account & Assets

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| GET | `/api/v1/trading/account` | Account overview per exchange | [→](../advanced-api-docs/get-portfolio-overview.md) |
| GET | `/api/v1/trading/portfolio/assets` | Portfolio asset breakdown | [→](../advanced-api-docs/get-portfolio-assets-details.md) |
| GET | `/api/v1/trading/user/tradingStats` | Trading statistics (`begin`, `end` params) | [→](../advanced-api-docs/query-user-tradingstats.md) |
| GET | `/api/v1/trading/userFeeRate` | Maker/Taker fee rates | [→](../advanced-api-docs/user-trading-fee-rate.md) |

### Orders

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| POST | `/api/v1/trading/order` | Place order | [→](../advanced-api-docs/place-order.md) |
| PUT | `/api/v1/trading/order` | Amend order | [→](../advanced-api-docs/replace-order.md) |
| DELETE | `/api/v1/trading/order` | Cancel order | [→](../advanced-api-docs/cancel-order.md) |
| DELETE | `/api/v1/trading/cancelAll` | Cancel all orders (`sym` or `exchangeType`) | [→](../advanced-api-docs/cancel-one-portfolio-orders.md) |
| GET | `/api/v1/trading/order` | Get single order (`orderId` or `clientOrderId`) | [→](../advanced-api-docs/query-order.md) |
| GET | `/api/v1/trading/orders` | List open orders | [→](../advanced-api-docs/current-open-orders.md) |
| GET | `/api/v1/trading/history/orders` | Order history (`sym` optional) | [→](../advanced-api-docs/order-history.md) |
| GET | `/api/v1/trading/archive/history/orders` | Archived order history | [→](../advanced-api-docs/order-history-archive.md) |
| GET | `/api/v1/trading/executions` | Execution records | [→](../advanced-api-docs/query-transactions.md) |
| GET | `/api/v1/trading/executions/pageable` | Pageable executions | [→](../advanced-api-docs/query-transactions-pageable.md) |
| GET | `/api/v1/trading/statement` | Trading statement | [→](../advanced-api-docs/query-statement.md) |

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

> For algo orders (TP/SL, TWAP, VWAP — conditional trigger orders), see [Algo Orders](../advanced-api-docs/algo-orders.md).

### Positions

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| GET | `/api/v1/trading/position` | Open positions (`sym`, `exchange` optional) | [→](../advanced-api-docs/query-portfolio-position.md) |
| GET | `/api/v1/trading/history/position` | Position history | [→](../advanced-api-docs/query-portfolio-history-position.md) |
| DELETE | `/api/v1/trading/position` | Close position (`sym`, `positionSide`) | [→](../advanced-api-docs/close-position.md) |
| DELETE | `/api/v1/trading/positions` | Close all positions (`exchangeType`, `closeAllPos:"true"`) | [→](../advanced-api-docs/close-positions.md) |
| GET | `/api/v1/trading/perp/leverage` | Get leverage (`sym` or `exchange` optional) | [→](../advanced-api-docs/get-perp-leverage.md) |
| POST | `/api/v1/trading/position/leverage` | Set leverage (`sym`, `leverage`) | [→](../advanced-api-docs/set-leverage.md) |
| GET | `/api/v1/adl/rank` | ADL rank (`sym` optional) | [→](../advanced-api-docs/query-adl-rank.md) |

### Market Data (RapidX)

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| GET | `/api/v1/trading/sym/info` | Symbol rules (`sym` optional) | [→](../advanced-api-docs/sym-info.md) |
| GET | `/api/v1/market/fundingRate` | Funding rate (`sym` required) | [→](../advanced-api-docs/get-current-fundingfee.md) |
| GET | `/api/v1/market/markPrice` | Mark price (`sym` optional) | [→](../advanced-api-docs/get-current-markprice.md) |
| GET | `/api/v1/trading/positionBracket` | Position tier/bracket | [→](../advanced-api-docs/positionbracket.md) |
| GET | `/api/v1/trading/loan/info` | Loan info | [→](../advanced-api-docs/get-loan-tier.md) |
| GET | `/api/v1/trading/coin/discount` | Coin discount rate | [→](../advanced-api-docs/get-discount-rate-detail.md) |
| GET | `/api/v1/trading/margin/leverage` | Margin leverage | [→](../advanced-api-docs/get-margin-leverage.md) |


### Rate Limits

| Endpoint | Competition limit |
|----------|------------------|
| POST `/api/v1/trading/order` (place order) | **1 req / 5 s** |
| PUT `/api/v1/trading/order` (replace order) | **1 req / 5 s** |
| DELETE `/api/v1/trading/order` (cancel order) | **1 req / 5 s** |
| GET `/api/v1/market/fundingRate` | **3 req / 10 s** |
| GET `/api/v1/market/markPrice` | **3 req / 10 s** |
| GET `/api/v1/trading/sym/info` | **3 req / 10 s** |
| DELETE `/api/v1/trading/positions` (close all positions) | **1 req / 10 s** |
| All other REST endpoints | production rate × 1/5 (see individual endpoint pages) |

> For full rate limits and endpoint details, see the official API documentation: https://apidocliquidity.readme.io/reference/place-order

### REST Response Format

```json
{
  "code": 200000,
  "message": "Success",
  "data": { ... }
}
```

`code: 200000` = success. Any other value is an error. [Error codes →](../advanced-api-docs/error-codes.md)

---

## WebSocket

### Connection Hosts

| Service | Phase I (Sandbox) | Phase II (Mainnet) |
|---------|-------------------|--------------------|
| Trading REST API | `https://api.ltp-contest.com` | `https://api.liquiditytech.com` |
| News Feed REST API | `https://api.ltp-contest.com` | `https://api.liquiditytech.com` |
| Market Data WebSocket | `wss://md.ltp-contest.com/marketdata/v2/public` | `wss://md.liquiditytech.com/marketdata/v2/public` |
| User Data WebSocket | `wss://wss.ltp-contest.com/v1/private` | `wss://wss.liquiditytech.com/v1/private` |
| News Feed WebSocket | `wss://feeds.ltp-contest.com/feeds/v2/public` | `wss://feeds.ltp-contest.com/feeds/v2/public` (unchanged across phases) |

> Use the Phase II (Mainnet) hosts once the main competition begins.

### Market Data WebSocket

[Market Data overview →](../advanced-api-docs/market-data-overview.md)

- **URL:** Phase II (Mainnet) `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox) `wss://md.ltp-contest.com/marketdata/v2/public`
- **Supported exchanges:** `BINANCE` / `OKX` / `EDX`
- **Compression:** GZIP is enabled by default. Append `?binary=false` to the URL to receive plain-text JSON frames. In Python, decompress GZIP frames with `zlib.decompress(data, zlib.MAX_WBITS | 16)`.

#### Authentication (optional)

Unauthenticated connections are subject to lower rate limits. To authenticate, send a login message after connecting:

```json
{
  "action": "login",
  "args": { "apiKey": "YOUR_ACCESS_KEY", "timestamp": "1778140847", "sign": "..." }
}
```

> **Field naming differs by message type:** login uses `action` / `args` (object); subscribe / unsubscribe use `event` / `arg` (array).
> **Timestamp:** the Market Data WS accepts both 10-digit (seconds) and 13-digit (milliseconds) timestamps. The private (User Data) WS only accepts seconds.

**Signature:**

```
message = timestamp + "GET" + "/users/self/verify"
sign = HMAC-SHA256(secret_key, message).hexdigest()
```

**Success response:**

```json
{ "event": "login", "code": "200000", "msg": "success" }
```

#### Rate Limits

| | Unauthenticated | Authenticated |
|---|---|---|
| Max connections per IP | 5 | 40 |
| Max trading pairs per connection | 5 | 50 |

- **Subscribe rate:** max 50 subscribe/unsubscribe requests per IP per second; exceeding returns error `11260`.
- Limits are counted by **trading pair**, not by channel — subscribing BBO + TICKER + TRADE for the same symbol counts as 1 pair.

#### Heartbeat

The server disconnects after 60 seconds of inactivity. Send `{"ping": <timestamp_ms>}` every ~20 seconds; the server replies `{"pong": <timestamp_ms>}`.

#### Symbol Format

`{EXCHANGE}_{TYPE}_{BASE}_{QUOTE}` — `EXCHANGE`: `BINANCE` / `OKX` / `EDX`; `TYPE`: `SPOT` / `PERP`. Example: `BINANCE_PERP_BTC_USDT`.

#### Subscribe / Unsubscribe

Channel names are case-sensitive and must be **uppercase**.

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO",   "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TRADE", "sym": "OKX_PERP_BTC_USDT" }
  ]
}
```

#### Available Channels

| Channel | Frequency | Scope | Docs |
|---------|-----------|-------|------|
| `TICKER` | 2000 ms | All | [→](../advanced-api-docs/market-data-ticker.md) |
| `ORDER_BOOK` | 250 ms | All | [→](../advanced-api-docs/market-data-order-book.md) |
| `BBO` | On change | All | [→](../advanced-api-docs/market-data-bbo.md) |
| `TRADE` | Real-time | All | [→](../advanced-api-docs/market-data-trade.md) |
| `MARK_PRICE` | Real-time | Perpetual only | [→](../advanced-api-docs/market-data-price.md) |
| `INDEX_PRICE` | Real-time | Perpetual only | [→](../advanced-api-docs/market-data-price.md) |
| `KLINE` | Real-time | All | [→](../advanced-api-docs/market-data-kline-candlestick.md) |
| `INDEX_KLINE` | Real-time | Perpetual only | [→](../advanced-api-docs/market-data-kline-candlestick.md) |
| `MARK_PRICE_KLINE` | Real-time | Perpetual only | [→](../advanced-api-docs/market-data-kline-candlestick.md) |
| `OPEN_INTEREST` | On change | Perpetual only | [→](../advanced-api-docs/market-data-open-interest.md) |

#### Error Codes

| Code | Meaning |
|------|---------|
| `200000` | Success |
| `11100` | Invalid symbol (e.g. `SPOT` on a perpetual-only channel) |
| `11101` | Invalid channel |
| `11102` | Invalid parameter |
| `11103` | Invalid request |
| `11106` | Invalid access key |
| `11107` | Invalid sign / signature |
| `11108` | Invalid timestamp |
| `11109` | Permission denied |
| `11112` | Not subscribed |
| `11200` | Too many connections per IP |
| `11210` | Unauthenticated symbol limit (5) exceeded |
| `11220` | Authenticated symbol limit (50) exceeded |
| `11230` | Connection not authenticated |
| `11240` | Subscribe rate limit exceeded |
| `11250` | Symbol not currently supported |
| `11260` | Request rate limit exceeded |

### User Data WebSocket

[User Data Streams overview →](../advanced-api-docs/ws-user-data-overview.md)

- **URL:** Phase II (Mainnet) `wss://wss.liquiditytech.com/v1/private` · Phase I (Sandbox) `wss://wss.ltp-contest.com/v1/private`
- **Supported exchanges:** `BINANCE` / `OKX` / `EDX`
- **Authentication required.** After login the server auto-pushes orders, trades, assets, positions, and margin-call updates; the same connection can place, replace, and cancel orders.

#### Login

```json
{
  "action": "login",
  "args": { "apiKey": "YOUR_ACCESS_KEY", "timestamp": "1778140847", "sign": "..." }
}
```

- **Timestamp:** only 10-digit seconds are accepted; a 13-digit millisecond timestamp returns `601016`.
- **Success:** `{"event":"login","code":0,"msg":""}`
- **Failure:** `{"event":"login","code":601009,"msg":"Login failed"}`
- **`args.onlyTrade`** (optional, bool): when `true`, the connection is restricted to order operations only and no data streams are pushed.

**Signature:**

```
message = timestamp + "GET" + "/users/self/verify"
sign = HMAC-SHA256(secret_key, message).hexdigest()
```

#### Order Actions

| Action | Docs |
|--------|------|
| `place_order` | [→](../advanced-api-docs/ws-place-order.md) |
| `replace_order` | [→](../advanced-api-docs/ws-replace-order.md) |
| `cancel_order` | [→](../advanced-api-docs/ws-cancel-order.md) |
| `cancel_orders` | [→](../advanced-api-docs/ws-cancel-orders.md) |

```json
{ "id": "p1", "action": "place_order",   "args": { ... } }
{ "id": "a1", "action": "replace_order", "args": { "orderId": "...", "replacePrice": "64000" } }
{ "id": "c1", "action": "cancel_order",  "args": { "orderId": "..." } }
```

**Orders push channel** — auto-subscribed after login. [User data streams →](../advanced-api-docs/ws-user-data-orders-trades-assets-positions.md)

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

#### Rate Limits

| Action | Limit |
|--------|-------|
| Login | 1 req/s per API key |
| Place Order | 1200 req / 60s |
| Replace Order | 300 req / 60s |
| Cancel Order | 1200 req / 60s |
| Cancel Orders (Batch) | 1200 req / 60s |

#### Heartbeat

The server disconnects after 30 seconds of inactivity. Send the raw string `ping` every ~10 seconds; the server replies `pong`.

#### Available Pages

- [User Data overview](../advanced-api-docs/ws-user-data-overview.md) · [User Data Streams](../advanced-api-docs/ws-user-data-orders-trades-assets-positions.md)
- [Place Order](../advanced-api-docs/ws-place-order.md) · [Replace Order](../advanced-api-docs/ws-replace-order.md)
- [Cancel Order](../advanced-api-docs/ws-cancel-order.md) · [Cancel Orders](../advanced-api-docs/ws-cancel-orders.md)

> For full rate limits and endpoint details, see the official API documentation: https://apidocliquidity.readme.io/reference/place-order

> For all News Feed APIs (REST + WebSocket), see [News Feed](04-news-feed.md).

---

## Symbol Format

```
{EXCHANGE}_{TYPE}_{BASE}_{QUOTE}
```

| Exchange | Type | Examples |
|----------|------|---------|
| `BINANCE` | `PERP` / `SPOT` | `BINANCE_PERP_BTC_USDT`, `BINANCE_SPOT_BTC_USDT` |
| `OKX` | `PERP` / `SPOT` | `OKX_PERP_BTC_USDT`, `OKX_SPOT_BTC_USDT` |
| `EDX` | `PERP` / `SPOT` | `EDX_PERP_BTC_USDT`, `EDX_SPOT_BTC_USDT` |

Channels marked "Perpetual only" reject `SPOT` symbols with error `11100`.

---

## Common Error Codes

[Full error code reference →](../advanced-api-docs/error-codes.md)

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

