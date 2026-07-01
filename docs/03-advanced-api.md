# Advanced: Direct REST & WebSocket API

> For custom code and low-level integrations that call RapidX directly, without the CLI or MCP layer.

**Most participants should use the CLI or MCP** — see [`01-quickstart.md`](./01-quickstart.md). This document is for cases where you need direct HTTP or WebSocket access: performance-critical bots, languages without Node.js, or custom tooling.

Official API docs: <https://apidocliquidity.readme.io/reference/get-account-list>

---

## Authentication

[Official docs →](https://apidocliquidity.readme.io/reference/authentication)

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

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| GET | `/api/v1/trading/account` | Account overview per exchange | [→](https://apidocliquidity.readme.io/reference/get-portfolio-overview) |
| GET | `/api/v1/trading/portfolio/assets` | Portfolio asset breakdown | [→](https://apidocliquidity.readme.io/reference/get-portfolio-assets-details) |
| GET | `/api/v1/trading/user/tradingStats` | Trading statistics (`begin`, `end` params) | [→](https://apidocliquidity.readme.io/reference/query-user-tradingstats) |
| GET | `/api/v1/trading/userFeeRate` | Maker/Taker fee rates | [→](https://apidocliquidity.readme.io/reference/user-trading-fee-rate) |

### Orders

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| POST | `/api/v1/trading/order` | Place order | [→](https://apidocliquidity.readme.io/reference/place-order) |
| PUT | `/api/v1/trading/order` | Amend order | [→](https://apidocliquidity.readme.io/reference/replace-order) |
| DELETE | `/api/v1/trading/order` | Cancel order | [→](https://apidocliquidity.readme.io/reference/cancel-order) |
| DELETE | `/api/v1/trading/cancelAll` | Cancel all orders (`sym` or `exchangeType`) | [→](https://apidocliquidity.readme.io/reference/cancel-one-portfolio-orders) |
| GET | `/api/v1/trading/order` | Get single order (`orderId` or `clientOrderId`) | [→](https://apidocliquidity.readme.io/reference/query-order) |
| GET | `/api/v1/trading/orders` | List open orders | [→](https://apidocliquidity.readme.io/reference/current-open-orders) |
| GET | `/api/v1/trading/history/orders` | Order history (`sym` optional) | [→](https://apidocliquidity.readme.io/reference/order-history) |
| GET | `/api/v1/trading/archive/history/orders` | Archived order history | [→](https://apidocliquidity.readme.io/reference/order-history-archive) |
| GET | `/api/v1/trading/executions` | Execution records | [→](https://apidocliquidity.readme.io/reference/query-transactions) |
| GET | `/api/v1/trading/executions/pageable` | Pageable executions | [→](https://apidocliquidity.readme.io/reference/query-transactions-pageable) |
| GET | `/api/v1/trading/statement` | Trading statement | [→](https://apidocliquidity.readme.io/reference/query-statement) |

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

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| GET | `/api/v1/trading/position` | Open positions (`sym`, `exchange` optional) | [→](https://apidocliquidity.readme.io/reference/query-portfolio-position) |
| GET | `/api/v1/trading/history/position` | Position history | [→](https://apidocliquidity.readme.io/reference/query-portfolio-history-position) |
| DELETE | `/api/v1/trading/position` | Close position (`sym`, `positionSide`) | [→](https://apidocliquidity.readme.io/reference/close-position) |
| DELETE | `/api/v1/trading/positions` | Close all positions (`exchangeType`, `closeAllPos:"true"`) | [→](https://apidocliquidity.readme.io/reference/close-positions) |
| GET | `/api/v1/trading/perp/leverage` | Get leverage (`sym` or `exchange` optional) | [→](https://apidocliquidity.readme.io/reference/get-perp-leverage) |
| POST | `/api/v1/trading/position/leverage` | Set leverage (`sym`, `leverage`) | [→](https://apidocliquidity.readme.io/reference/set-leverage) |
| GET | `/api/v1/adl/rank` | ADL rank (`sym` optional) | [→](https://apidocliquidity.readme.io/reference/query-adl-rank) |

### Market Data (RapidX)

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| GET | `/api/v1/trading/sym/info` | Symbol rules (`sym` optional) | [→](https://apidocliquidity.readme.io/reference/sym-info) |
| GET | `/api/v1/market/fundingRate` | Funding rate (`sym` required) | [→](https://apidocliquidity.readme.io/reference/get-current-fundingfee) |
| GET | `/api/v1/market/markPrice` | Mark price (`sym` optional) | [→](https://apidocliquidity.readme.io/reference/get-current-markprice) |
| GET | `/api/v1/trading/positionBracket` | Position tier/bracket | [→](https://apidocliquidity.readme.io/reference/positionbracket) |
| GET | `/api/v1/trading/loan/info` | Loan info | [→](https://apidocliquidity.readme.io/reference/get-loan-tier) |
| GET | `/api/v1/trading/coin/discount` | Coin discount rate | [→](https://apidocliquidity.readme.io/reference/get-discount-rate-detail) |
| GET | `/api/v1/trading/margin/leverage` | Margin leverage | [→](https://apidocliquidity.readme.io/reference/get-margin-leverage) |

### REST Response Format

```json
{
  "code": 200000,
  "message": "Success",
  "data": { ... }
}
```

`code: 200000` = success. Any other value is an error. [Error codes →](https://apidocliquidity.readme.io/reference/error-codes)

---

## WebSocket

### Private WebSocket (Trading + Account Events)

[User Data Streams overview →](https://apidocliquidity.readme.io/reference/ws-user-data-overview)

**URL:** `wss://wss.ltp-contest.com/v1/private`

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

| Action | Docs |
|--------|------|
| `place_order` | [→](https://apidocliquidity.readme.io/reference/ws-place-order) |
| `replace_order` | [→](https://apidocliquidity.readme.io/reference/ws-replace-order) |
| `cancel_order` | [→](https://apidocliquidity.readme.io/reference/ws-cancel-order) |
| `cancel_orders` | [→](https://apidocliquidity.readme.io/reference/ws-cancel-orders) |

```json
{ "id": "p1", "action": "place_order",  "args": { ... } }
{ "id": "a1", "action": "replace_order","args": { "orderId": "...", "replacePrice": "64000" } }
{ "id": "c1", "action": "cancel_order", "args": { "orderId": "..." } }
```

**Orders push channel** — auto-subscribed after login. [User data streams →](https://apidocliquidity.readme.io/reference/ws-user-data-orders-trades-assets-positions)

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

[Market Data overview →](https://apidocliquidity.readme.io/reference/market-data-overview)

**URL:** `wss://mds.ltp-contest.com/marketdata/v2/public`

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

| Channel | Frequency | Scope | Docs |
|---------|-----------|-------|------|
| `BBO` | On change | All | [→](https://apidocliquidity.readme.io/reference/market-data-bbo) |
| `TICKER` | 2000 ms | All | [→](https://apidocliquidity.readme.io/reference/market-data-ticker) |
| `TRADE` | Real-time | All | [→](https://apidocliquidity.readme.io/reference/market-data-trade) |
| `ORDER_BOOK` | 250 ms | All | [→](https://apidocliquidity.readme.io/reference/market-data-order-book) |
| `KLINE` | Real-time | All | [→](https://apidocliquidity.readme.io/reference/market-data-kline-candlestick) |
| `MARK_PRICE` | Real-time | Perpetual only | [→](https://apidocliquidity.readme.io/reference/market-data-price) |
| `INDEX_PRICE` | Real-time | Perpetual only | [→](https://apidocliquidity.readme.io/reference/market-data-price) |
| `MARK_PRICE_KLINE` | Real-time | Perpetual only | [→](https://apidocliquidity.readme.io/reference/market-data-kline-candlestick) |
| `INDEX_KLINE` | Real-time | Perpetual only | [→](https://apidocliquidity.readme.io/reference/market-data-kline-candlestick) |
| `OPEN_INTEREST` | On change | Perpetual only | [→](https://apidocliquidity.readme.io/reference/market-data-open-interest) |

Keepalive: send `{ "ping": <timestamp_ms> }` every 20 seconds; server replies `{ "pong": <timestamp_ms> }`.

---

## News Feed WebSocket

Real-time crypto news and hot-topic push stream sourced from the feeds-collect-service.

### Endpoint & Authentication

**URL:** `wss://feeds.ltp-contest.com/feeds/v2/public`

No authentication required. Connect and subscribe immediately — the endpoint is public.

### Connection Limits

| Limit | Value | Notes |
|-------|-------|-------|
| Max concurrent connections / IP | **5** | Exceeding this returns HTTP **429** at the WS upgrade handshake |
| Heartbeat timeout | **90 seconds** | Server closes the connection if no `ping` is received within 90 s |
| Heartbeat check cycle | 30 seconds | Server scans for stale connections every 30 s |

IP is identified from `X-Forwarded-For` (first segment) → `X-Real-IP` → RemoteAddress.

### Subscribe

Send a `subscribe` message immediately after connecting. Multiple channels can be batched in one request; subscribing to the same channel more than once is idempotent.

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "news.category.all" },
    { "channel": "news.hot.all" }
  ]
}
```

**Subscribe response:**

```json
{
  "event": "subscribe",
  "code": "0",
  "msg": "success",
  "arg": [
    { "channel": "news.category.all" },
    { "channel": "news.hot.all" }
  ],
  "invalidArg": []
}
```

| Field | Description |
|-------|-------------|
| `code` | `"0"` = success |
| `arg` | Channels that were successfully subscribed |
| `invalidArg` | Unrecognised channels — does not affect valid subscriptions |

### Channels

| Channel | Description | Push trigger |
|---------|-------------|--------------|
| `news.category.all` | All-category crypto news items | New article ingested by feeds-collect-service |
| `news.hot.all` | Hot-topic items | New hot item ingested by feeds-collect-service |

Pushes are fan-out broadcasts — every connection subscribed to the same channel receives the identical message. There is no per-connection filtering.

### Message Structures

#### News push — `news.category.all`

```json
{
  "channel": "news.category.all",
  "data": {
    "id": 100001,
    "newsId": "tweet_1234567890",
    "newsType": "news",
    "category": 1,
    "title": "BTC surges above $70,000",
    "content": "Bitcoin has broken the $70k resistance...",
    "publishTime": 1719388800000,
    "sourceUrl": "https://twitter.com/xxx/status/xxx",
    "originalUrl": "https://twitter.com/xxx/status/xxx",
    "authorName": "cryptonews",
    "authorDisplayName": "Crypto News",
    "authorAvatar": "https://cdn.example.com/avatar.jpg",
    "isBlueVerified": true,
    "viewCount": 12000,
    "likeCount": 3400,
    "commentCount": 210,
    "retweetCount": 890,
    "currencies": [
      { "currencyId": "1", "fullName": "BITCOIN", "symbol": "BTC" }
    ],
    "mediaAttachments": [
      {
        "sosoUrl": "https://cdn.sosovalue.com/media/xxx.jpg",
        "originalUrl": "https://pbs.twimg.com/media/xxx.jpg",
        "shortUrl": null,
        "mediaType": "photo"
      }
    ],
    "quotedContent": {
      "content": "Original tweet being quoted...",
      "createdAtSource": 1719388700000,
      "originalUrl": "https://twitter.com/yyy/status/yyy",
      "author": "another_user",
      "nickName": "Another User",
      "authorAvatarUrl": "https://cdn.example.com/avatar2.jpg",
      "isBlueVerified": false,
      "verifiedType": null,
      "impressionCount": 5000,
      "likeCount": 120,
      "replyCount": 30,
      "retweetCount": 45,
      "mediaAttachments": []
    },
    "createdAt": 1719388810000
  }
}
```

**`data` fields:**

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `id` | long | No | Database primary key |
| `newsId` | string | No | Business unique ID (e.g. original tweet ID) |
| `newsType` | string | No | Fixed value `"news"` |
| `category` | int | Yes | Category ID |
| `title` | string | Yes | Headline |
| `content` | string | Yes | Body text |
| `publishTime` | long | Yes | Original publish time (epoch ms) |
| `sourceUrl` | string | Yes | Source link |
| `originalUrl` | string | Yes | Original article link |
| `authorName` | string | Yes | Author username |
| `authorDisplayName` | string | Yes | Author display name |
| `authorAvatar` | string | Yes | Author avatar URL |
| `isBlueVerified` | boolean | Yes | Blue-tick verified |
| `viewCount` | long | Yes | View count |
| `likeCount` | long | Yes | Like count |
| `commentCount` | long | Yes | Comment count |
| `retweetCount` | long | Yes | Retweet count |
| `currencies` | array\|null | Yes | Associated currencies — see `currencies` below |
| `mediaAttachments` | array\|null | Yes | Media attachments — see `mediaAttachments` below |
| `quotedContent` | object\|null | Yes | Quoted tweet — see `quotedContent` below |
| `createdAt` | long | No | Ingestion time (epoch ms) |

**`currencies` items:**

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `currencyId` | string | No | Platform currency ID |
| `fullName` | string | Yes | Full name, e.g. `BITCOIN` |
| `symbol` | string | Yes | Ticker symbol, e.g. `BTC` |

**`mediaAttachments` items:**

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `sosoUrl` | string | No | CDN-hosted URL |
| `originalUrl` | string | Yes | Original media URL |
| `shortUrl` | string | Yes | Short URL |
| `mediaType` | string | No | `photo` / `video` / `gif` |

**`quotedContent` fields:**

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `content` | string | Yes | Quoted text |
| `createdAtSource` | long | Yes | Quoted tweet publish time (epoch ms) |
| `originalUrl` | string | Yes | Quoted tweet link |
| `author` | string | Yes | Twitter @username |
| `nickName` | string | Yes | Author display name |
| `authorAvatarUrl` | string | Yes | Author avatar URL |
| `isBlueVerified` | boolean | Yes | Blue-tick verified |
| `verifiedType` | string | Yes | Verification type, e.g. `Business` |
| `impressionCount` | long | Yes | View count |
| `likeCount` | long | Yes | Like count |
| `replyCount` | long | Yes | Reply count |
| `retweetCount` | long | Yes | Retweet count |
| `mediaAttachments` | array\|null | Yes | Same structure as parent `mediaAttachments` |

#### Hot push — `news.hot.all`

```json
{
  "channel": "news.hot.all",
  "data": {
    "id": 200001,
    "newsId": "hot_9876543210",
    "newsType": "hot",
    "title": "Ethereum upgrade scheduled",
    "content": "The next major Ethereum upgrade...",
    "publishTime": 1719388800000,
    "ingestTime": 1719388820000,
    "sourceUrl": "https://example.com/news/xxx",
    "createdAt": 1719388825000
  }
}
```

Hot items contain a reduced field set compared to news items:

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `id` | long | No | Database primary key |
| `newsId` | string | No | Business unique ID |
| `newsType` | string | No | Fixed value `"hot"` |
| `title` | string | Yes | Headline |
| `content` | string | Yes | Body text |
| `publishTime` | long | Yes | Original publish time (epoch ms) |
| `ingestTime` | long | No | Ingestion time (epoch ms) |
| `sourceUrl` | string | Yes | Source link |
| `createdAt` | long | No | Database insert time (epoch ms) |

> Hot items do **not** include `category`, `author*`, interaction counts, `currencies`, `mediaAttachments`, or `quotedContent`.

#### Heartbeat

Client must send a ping at least every 90 seconds. Recommended interval: ≤ 30 seconds.

```json
{ "ping": 1719388800000 }
```

Server response:

```json
{ "pong": 1719388800001 }
```

### Error Codes

| Layer | Code | Condition |
|-------|------|-----------|
| HTTP | `429` | IP concurrent connection limit (5) exceeded at WS handshake |
| WS message | `500` | Unexpected server error processing a client message |

Error message structure:

```json
{ "event": "error", "code": "500", "msg": "Server Error" }
```

### Python Quick-Start

```python
import asyncio, json, time, ssl
import websockets
from websockets.exceptions import ConnectionClosed

URL = "wss://feeds.ltp-contest.com/feeds/v2/public"
CHANNELS = ["news.category.all", "news.hot.all"]
HEARTBEAT_INTERVAL = 20  # seconds (server timeout: 90 s)


async def on_push(channel: str, data: dict) -> None:
    if channel == "news.category.all":
        print(f"[news] {data.get('newsId')}  {data.get('title')}")
    elif channel == "news.hot.all":
        print(f"[hot]  {data.get('newsId')}  {data.get('title')}")


async def heartbeat(ws) -> None:
    while True:
        await asyncio.sleep(HEARTBEAT_INTERVAL)
        await ws.send(json.dumps({"ping": int(time.time() * 1000)}))


async def run() -> None:
    reconnect_delay = 1.0
    while True:
        try:
            async with websockets.connect(URL) as ws:
                print("[feeds] connected")
                reconnect_delay = 1.0

                await ws.send(json.dumps({
                    "event": "subscribe",
                    "arg": [{"channel": c} for c in CHANNELS]
                }))

                hb = asyncio.create_task(heartbeat(ws))
                try:
                    async for raw in ws:
                        msg = json.loads(raw)
                        if "pong" in msg:
                            continue
                        if msg.get("event") == "subscribe":
                            print("[feeds] subscribed:", msg.get("arg"),
                                  "invalid:", msg.get("invalidArg"))
                        elif msg.get("event") == "error":
                            print("[feeds] error:", msg.get("code"), msg.get("msg"))
                        else:
                            await on_push(msg.get("channel", ""), msg.get("data", {}))
                finally:
                    hb.cancel()

        except ConnectionClosed as e:
            print(f"[feeds] closed: {e}")
        except Exception as e:
            print(f"[feeds] error: {e}")

        print(f"[feeds] reconnect in {reconnect_delay:.0f}s")
        await asyncio.sleep(reconnect_delay)
        reconnect_delay = min(reconnect_delay * 2, 30.0)


if __name__ == "__main__":
    asyncio.run(run())
```

### Notes

1. **Heartbeat is client-initiated.** Send `ping` every ≤ 30 seconds. The server closes the connection after 90 seconds of inactivity.
2. **IP limit is 5 concurrent connections.** HTTP 429 is returned at the WS handshake if exceeded. Implement connection reuse or exponential backoff.
3. **Subscription state is not preserved on reconnect.** Re-send the `subscribe` message after every reconnection.
4. **Batch subscriptions are supported.** A single `subscribe` message may list multiple channels.
5. **Optional fields may be `null`.** Always null-check `currencies`, `mediaAttachments`, and `quotedContent` before accessing their contents.
6. **Pushes are fan-out broadcasts.** All connections subscribed to the same channel receive the same message — there is no per-connection filtering.

---

## Symbol Format

```
{EXCHANGE}_{TYPE}_{BASE}_{QUOTE}
```

| Exchange | Type | Examples |
|----------|------|---------|
| `BINANCE` | `PERP` | `BINANCE_PERP_BTC_USDT`, `BINANCE_PERP_ETH_USDT` |

Channels marked "Perpetual only" reject `SPOT` symbols with error `11100`.

---

## Common Error Codes

[Full error code reference →](https://apidocliquidity.readme.io/reference/error-codes)

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

