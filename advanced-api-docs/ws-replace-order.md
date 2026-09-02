# WebSocket — Replace Order (Amend)

**WebSocket URL:** Phase II (Mainnet): `wss://wss.liquiditytech.com/v1/private` · Phase I (Sandbox): `wss://wss.ltp-contest.com/v1/private`
> **Official API docs:** https://apidocliquidity.readme.io/reference/ws-replace-order

Amend the price and/or quantity of a resting open order.

**Rate limit**: 1 request / 5 s

## Request

```json
{
  "id": "replace-1",
  "action": "replace_order",
  "args": {
    "orderId":      "1234567890123456",
    "replacePrice": "80000"
  }
}
```

## Request Fields

| Field | Required | Description |
|-------|----------|-------------|
| `orderId` | Conditional | System order ID (or `clientOrderId`) |
| `clientOrderId` | Conditional | Client order ID (or `orderId`) |
| `replacePrice` | At least one | New price |
| `replaceQty` | At least one | New quantity |

## Response (success)

```json
{
  "id":    "replace-1",
  "event": "replace_order",
  "code":  200000,
  "msg":   "Success",
  "data": {
    "orderId":    "1234567890123456",
    "orderState": "OPEN"
  }
}
```

> SPOT and some PERP amends use cancel-then-replace internally. If `orderState` is `CANCELLED` or `REJECTED`, the amend failed and the original order is gone.
