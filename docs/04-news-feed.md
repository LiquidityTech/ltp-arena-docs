# News Feed API

Real-time and historical crypto news, hot topics, and featured articles from the LTP feeds service.

---

## Overview

The News Feed service exposes two integration paths:

| Path | When to use |
|------|-------------|
| **REST API** (Phase I: `https://api.ltp-contest.com` · Phase II: `https://api.liquiditytech.com`) | Historical or time-range queries; batch retrieval; backfilling |
| **WebSocket** (`wss://feeds.ltp-contest.com/feeds/v2/public`) | Real-time push for new articles as they are ingested; no polling required |

Both paths share the same data model. REST returns paginated snapshots; WebSocket delivers incremental pushes as events occur.

---

## Authentication

### V2 — Header Signature (recommended for REST)

Same header scheme as the trading API.

| Header | Value |
|--------|-------|
| `X-MBX-APIKEY` | Your Access Key |
| `nonce` | Current Unix timestamp in **seconds** (string) |
| `signature` | HMAC-SHA256 signature (see below) |

**Signature algorithm:**

```
1. Sort all query parameters alphabetically by key:
   sorted_payload = "key1=val1&key2=val2&..."

2. Append "&" + nonce:
   sign_string = sorted_payload + "&" + nonce

3. HMAC-SHA256 with Secret Key, hex-encoded:
   signature = HMAC-SHA256(secret_key, sign_string).hexdigest()
```

Server allows ±60 seconds clock skew on `nonce`.

### V1 — Query-Parameter Signature (alternative)

Credentials are passed as query parameters instead of headers.

| Query parameter | Description |
|----------------|-------------|
| `apiKey` | Your Access Key |
| `timestamp` | UTC datetime string, format `yyyy-MM-dd'T'HH:mm:ss`, e.g. `2026-07-02T03:30:00` |
| `sign` | HMAC-SHA256 signature (see below) |

**Signature algorithm:**

```
1. Collect all request parameters including apiKey and timestamp,
   but excluding sign and any parameters with empty values.
2. Sort by key alphabetically.
3. Join as "key1=value1&key2=value2&..."
4. HMAC-SHA256 with Secret Key, hex-encoded.
```

Server allows ±5 minutes clock skew on `timestamp`.

> **Key difference**: V2 uses Unix-second `nonce` in headers; V1 uses a formatted UTC string `timestamp` in query params. The HMAC inputs differ accordingly. Both are accepted by all three endpoints.

---

## REST API

Base URL: Phase I (Sandbox) `https://api.ltp-contest.com` · Phase II (Mainnet) `https://api.liquiditytech.com`

### Response Envelope

All Feeds REST endpoints return a paginated JSON envelope. Note: the success code is `200`, not `200000` as used by the trading API.

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "page": 1,
    "pageSize": 20,
    "pageNum": 5,
    "totalSize": 98,
    "list": [ ... ]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | int | `200` = success; any other value is an error |
| `message` | string | Status description |
| `data.page` | int | Current page number |
| `data.pageSize` | int | Items per page |
| `data.pageNum` | int | Total number of pages |
| `data.totalSize` | int | Total item count |
| `data.list` | array | Payload items |

### GET /api/v1/feeds/queryNews

Returns paginated crypto news articles. **Data retention: 15 days.** `startTime` values older than the retention boundary are automatically clamped to the boundary without error.

**Query parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `startTime` | long | No | Start time (epoch ms) |
| `endTime` | long | No | End time (epoch ms) |
| `categories` | string | No | Comma-separated category IDs, e.g. `1,2,3` — see [Category Values](#category-values) |
| `page` | int | No | Page number, default `1` |
| `pageSize` | int | No | Items per page, default `20` |

**Response `list` item (NewsVO):**

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `id` | long | No | Database primary key |
| `newsId` | string | No | Source-side article ID |
| `newsType` | string | No | Fixed value `"news"` |
| `category` | int | Yes | Category ID — see [Category Values](#category-values) |
| `title` | string | Yes | English headline |
| `content` | string | Yes | Body HTML |
| `publishTime` | long | Yes | Original publish time (epoch ms) |
| `sourceUrl` | string | Yes | Platform article link |
| `originalUrl` | string | Yes | Original media link (may be null) |
| `authorName` | string | Yes | Twitter @username |
| `authorDisplayName` | string | Yes | Author display name |
| `authorAvatar` | string | Yes | Author avatar URL |
| `isBlueVerified` | boolean | Yes | Blue-tick verified |
| `viewCount` | long | Yes | View count |
| `likeCount` | long | Yes | Like count |
| `commentCount` | long | Yes | Comment count |
| `retweetCount` | long | Yes | Retweet count |
| `currencies` | array\|null | Yes | Associated currencies — see [CurrencyVO](#currencyvo) |
| `mediaAttachments` | array\|null | Yes | Media attachments — see [MediaAttachmentVO](#mediaattachmentvo) |
| `quotedContent` | object\|null | Yes | Quoted tweet content — see [QuoteContentVO](#quotecontentvo) |
| `createdAt` | long | No | Ingestion time (epoch ms) |

---

### GET /api/v1/feeds/queryHot

Returns paginated hot-topic items. **Data retention: 7 days.**

**Query parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `startTime` | long | No | Start time (epoch ms) |
| `endTime` | long | No | End time (epoch ms) |
| `page` | int | No | Page number, default `1` |
| `pageSize` | int | No | Items per page, default `20` |

**Response `list` item (HotNewsVO):**

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `id` | long | No | Database primary key |
| `newsId` | string | No | Source-side hot-topic ID |
| `newsType` | string | No | Fixed value `"hot"` |
| `title` | string | Yes | Aggregated English headline |
| `content` | string | Yes | Body HTML |
| `publishTime` | long | Yes | Original publish time (epoch ms) |
| `ingestTime` | long | No | Ingestion time (epoch ms) |
| `sourceUrl` | string | Yes | Platform article link |
| `createdAt` | long | No | Database insert time (epoch ms) |

> Hot items do **not** include `category`, `author*`, interaction counts, `currencies`, `mediaAttachments`, or `quotedContent`.

---

### GET /api/v1/feeds/queryFeatured

Returns paginated featured/curated articles. **Data retention: 180 days.**

**Query parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `startTime` | long | No | Start time (epoch ms) |
| `endTime` | long | No | End time (epoch ms) |
| `categories` | string | No | Comma-separated category IDs — see [Category Values](#category-values) |
| `page` | int | No | Page number, default `1` |
| `pageSize` | int | No | Items per page, default `20` |

**Response `list` item (FeaturedNewsVO):**

Same fields as NewsVO, with `newsType` fixed to `"featured"`. The `ingestTime` field is not present.

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `id` | long | No | Database primary key |
| `newsId` | string | No | Source-side article ID |
| `newsType` | string | No | Fixed value `"featured"` |
| `category` | int | Yes | Category ID |
| `title` | string | Yes | English headline |
| `content` | string | Yes | Body HTML |
| `publishTime` | long | Yes | Original publish time (epoch ms) |
| `sourceUrl` | string | Yes | Platform article link |
| `originalUrl` | string | Yes | Original media link |
| `authorName` | string | Yes | Twitter @username |
| `authorDisplayName` | string | Yes | Author display name |
| `authorAvatar` | string | Yes | Author avatar URL |
| `isBlueVerified` | boolean | Yes | Blue-tick verified |
| `currencies` | array\|null | Yes | Associated currencies — see [CurrencyVO](#currencyvo) |
| `mediaAttachments` | array\|null | Yes | Media attachments — see [MediaAttachmentVO](#mediaattachmentvo) |
| `quotedContent` | object\|null | Yes | Quoted tweet content — see [QuoteContentVO](#quotecontentvo) |
| `createdAt` | long | No | Ingestion time (epoch ms) |

---

### Category Values

Used in the `categories` filter parameter and the `category` response field.

| ID | Label |
|----|-------|
| `1` | News |
| `2` | Research |
| `3` | Institutional |
| `4` | KOL Insights |
| `7` | Announcements |
| `13` | Crypto Stock News |

---

### Nested Object Schemas

#### CurrencyVO

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `currencyId` | string | No | Platform currency ID |
| `fullName` | string | Yes | Full name, e.g. `BITCOIN` |
| `symbol` | string | Yes | Ticker symbol, e.g. `BTC` |

#### MediaAttachmentVO

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `sosoUrl` | string | No | CDN-hosted URL |
| `originalUrl` | string | Yes | Original media URL |
| `shortUrl` | string | Yes | Short URL |
| `mediaType` | string | No | `photo` / `video` / `gif` |

#### QuoteContentVO

| Field | Type | Nullable | Description |
|-------|------|---------|-------------|
| `content` | string | Yes | Quoted tweet body |
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
| `mediaAttachments` | array\|null | Yes | Same structure as [MediaAttachmentVO](#mediaattachmentvo) |

---

### REST Error Codes

| Code | Meaning |
|------|---------|
| `200` | Success |
| `1003` | API Key does not exist |
| `1004` | Signature error |
| `1007` | IP not on allowlist |
| `1010` | KYC level insufficient |
| `2003` | No access permission |

---

## WebSocket

Real-time push for new articles and hot topics as they are ingested — no polling required.

### Endpoint & Authentication

**URL:** `wss://feeds.ltp-contest.com/feeds/v2/public`

No authentication required. Connect and subscribe immediately — the endpoint is public.

### Connection Limits

| Limit | Value | Notes |
|-------|-------|-------|
| Max concurrent connections / IP | **5** | Exceeding returns HTTP **429** at the WS handshake |
| Heartbeat timeout | **90 seconds** | Server closes connection if no `ping` received |
| Heartbeat check cycle | 30 seconds | Server scans for stale connections every 30 s |

IP is identified from `X-Forwarded-For` (first segment) → `X-Real-IP` → RemoteAddress.

### Subscribe

Send a `subscribe` message immediately after connecting. Multiple channels can be batched; re-subscribing the same channel is idempotent.

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
| `arg` | Successfully subscribed channels |
| `invalidArg` | Unrecognised channels — does not affect valid subscriptions |

### Channels

| Channel | Description | Push trigger |
|---------|-------------|--------------|
| `news.category.all` | All-category crypto news items | New article ingested |
| `news.hot.all` | Hot-topic items | New hot item ingested |

Pushes are fan-out broadcasts — all connections subscribed to the same channel receive the same message.

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
    "quotedContent": null,
    "createdAt": 1719388810000
  }
}
```

Field schema follows [NewsVO](#get-apiv1feedsquerynews). All nested object schemas are identical to the REST API.

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

Field schema follows [HotNewsVO](#get-apiv1feedsqueryhot).

#### Heartbeat

```json
{ "ping": 1719388800000 }
```

Server response:

```json
{ "pong": 1719388800001 }
```

### WebSocket Error Codes

| Layer | Code | Condition |
|-------|------|-----------|
| HTTP | `429` | IP concurrent connection limit (5) exceeded at WS handshake |
| WS message | `500` | Unexpected server error |

Error message structure: `{ "event": "error", "code": "500", "msg": "Server Error" }`

---

## Python Quick-Start

### REST (V2 signature)

```python
import hashlib, hmac, time
import requests

API_KEY    = "<your-access-key>"
SECRET_KEY = "<your-secret-key>"
BASE       = "https://api.liquiditytech.com"  # Phase II (Mainnet); use https://api.ltp-contest.com for Phase I (Sandbox)


def _sign_v2(params: dict, nonce: int) -> str:
    s = "&".join(f"{k}={v}" for k, v in sorted(params.items())) + f"&{nonce}"
    return hmac.new(SECRET_KEY.encode(), s.encode(), hashlib.sha256).hexdigest()


def feeds_get(path: str, params: dict = {}) -> dict:
    nonce = int(time.time())
    headers = {
        "X-MBX-APIKEY": API_KEY,
        "nonce": str(nonce),
        "signature": _sign_v2(params, nonce),
    }
    return requests.get(BASE + path, params=params, headers=headers, timeout=10).json()


if __name__ == "__main__":
    import json

    # Latest news (page 1, 10 items)
    result = feeds_get("/api/v1/feeds/queryNews", {"page": "1", "pageSize": "10"})
    print(json.dumps(result, indent=2))

    # Hot topics from the last hour
    now_ms = int(time.time() * 1000)
    result = feeds_get("/api/v1/feeds/queryHot", {
        "startTime": str(now_ms - 3_600_000),
        "endTime": str(now_ms),
        "page": "1", "pageSize": "20",
    })
    print(json.dumps(result, indent=2))

    # Featured news filtered by category (News + Research)
    result = feeds_get("/api/v1/feeds/queryFeatured", {
        "categories": "1,2", "page": "1", "pageSize": "20",
    })
    print(json.dumps(result, indent=2))
```

### WebSocket

```python
import asyncio, json, time
import websockets
from websockets.exceptions import ConnectionClosed

URL      = "wss://feeds.ltp-contest.com/feeds/v2/public"
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
    delay = 1.0
    while True:
        try:
            async with websockets.connect(URL) as ws:
                print("[feeds] connected")
                delay = 1.0
                await ws.send(json.dumps({
                    "event": "subscribe",
                    "arg": [{"channel": c} for c in CHANNELS],
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
        print(f"[feeds] reconnect in {delay:.0f}s")
        await asyncio.sleep(delay)
        delay = min(delay * 2, 30.0)


if __name__ == "__main__":
    asyncio.run(run())
```

---

## Notes

1. **Time parameters** use epoch milliseconds (13-digit), e.g. `1751000000000`.
2. **Retention clamping**: `startTime` older than the retention window is silently clamped to the boundary — no error is returned.
3. **V1 timestamp format**: must be UTC, formatted as `yyyy-MM-dd'T'HH:mm:ss`, e.g. `2026-07-02T03:30:00`. Server allows ±5 minutes skew.
4. **V2 nonce**: Unix seconds. Server allows ±60 seconds skew. Keep system clock accurate.
5. **Optional fields may be `null`**: always null-check `currencies`, `mediaAttachments`, and `quotedContent`.
6. **WebSocket subscription state is not preserved** on reconnect — re-send the `subscribe` message after every reconnection.
