# WebSocket — Place Order

**WebSocket URL:** Phase II (Mainnet): `wss://wss.liquiditytech.com/v1/private` · Phase I (Sandbox): `wss://wss.ltp-contest.com/v1/private`
> **Official API docs:** https://apidocliquidity.readme.io/reference/ws-place-order

Submit a new order over the private WebSocket. Requires prior login.

**Rate limit**: 1 request / 5 s

## Request

```json
{
  "id": "place-1",
  "action": "place_order",
  "args": {
    "clientOrderId": "agent-001",
    "sym":           "BINANCE_PERP_BTC_USDT",
    "side":          "BUY",
    "positionSide":  "LONG",
    "orderType":     "LIMIT",
    "timeInForce":   "GTC",
    "orderQty":      "0.001",
    "limitPrice":    "81000"
  }
}
```

## Request Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | No | Client-assigned message ID — echoed in the response |
| `clientOrderId` | Recommended | Idempotency key, max 40 chars |
| `sym` | Yes | Symbol, e.g. `BINANCE_PERP_BTC_USDT` |
| `side` | Yes | `BUY` / `SELL` |
| `positionSide` | Yes (hedge mode) | `LONG` / `SHORT` |
| `orderType` | Yes | `LIMIT` / `MARKET` |
| `timeInForce` | For LIMIT | `GTC` / `IOC` / `FOK` / `GTX` |
| `orderQty` | Yes | Base/contract quantity |
| `limitPrice` | For LIMIT | Limit price |

## Response (success)

```json
{
  "id":    "place-1",
  "event": "place_order",
  "code":  200000,
  "msg":   "Success",
  "data": {
    "orderId":       "1234567890123456",
    "clientOrderId": "agent-001"
  }
}
```

> Submission success ≠ filled. Listen to the `Orders` push channel (see [ws-user-data-orders-trades-assets-positions.md](ws-user-data-orders-trades-assets-positions.md)) to confirm final state.
