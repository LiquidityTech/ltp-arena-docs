# User Data Streams — Overview

**WebSocket URL:** Phase II (Mainnet): `wss://wss.liquiditytech.com/v1/private` · Phase I (Sandbox): `wss://wss.ltp-contest.com/v1/private`
> **Official API docs:** https://apidocliquidity.readme.io/reference/ws-user-data-overview

The private WebSocket delivers real-time order, position, and account updates. Authentication is required.

## Authentication

After connecting, send a login message before subscribing or sending any commands:

```json
{
  "action": "login",
  "args": {
    "apiKey":    "YOUR_ACCESS_KEY",
    "timestamp": "1719388800",
    "sign":      "<HMAC-SHA256 hex>"
  }
}
```

**Signature:** `HMAC-SHA256(secret_key, timestamp + "GET" + "/users/self/verify")`

**Login response (success):**

```json
{ "event": "login", "code": 0, "msg": "" }
```

## Auto-Subscribed Channels

After a successful login the following push channels activate automatically:

| Channel | Content |
|---------|---------|
| `Orders` | Order state changes (new, fill, cancel, etc.) |
| `Positions` | Position updates |
| `Assets` | Account balance changes |

## Keepalive

Send `"ping"` every 15 seconds; server responds with `"pong"`.

## Order Action Commands

| Action | Description |
|--------|-------------|
| [`place_order`](ws-place-order.md) | Submit a new order |
| [`replace_order`](ws-replace-order.md) | Amend price/quantity of an open order |
| [`cancel_order`](ws-cancel-order.md) | Cancel a single open order |
| [`cancel_orders`](ws-cancel-orders.md) | Cancel all open orders for a symbol |

## Orders Push Channel

```json
{
  "channel": "Orders",
  "data": {
    "orderId":          "1234567890123456",
    "clientOrderId":    "agent-001",
    "sym":              "BINANCE_PERP_BTC_USDT",
    "side":             "BUY",
    "orderState":       "FILLED",
    "executedQty":      "0.001",
    "executedAvgPrice": "81422.4",
    "executedAmount":   "81.422"
  }
}
```

