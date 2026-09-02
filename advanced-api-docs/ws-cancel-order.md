# WebSocket — Cancel Order

**WebSocket URL:** Phase II (Mainnet): `wss://wss.liquiditytech.com/v1/private` · Phase I (Sandbox): `wss://wss.ltp-contest.com/v1/private`
> **Official API docs:** https://apidocliquidity.readme.io/reference/ws-cancel-order

Cancel a single open order.

**Rate limit**: 1 request / 5 s

## Request

```json
{
  "id": "cancel-1",
  "action": "cancel_order",
  "args": {
    "orderId": "1234567890123456"
  }
}
```

## Request Fields

| Field | Required | Description |
|-------|----------|-------------|
| `orderId` | Conditional | System order ID (or `clientOrderId`) |
| `clientOrderId` | Conditional | Client order ID (or `orderId`) |

## Response (accepted)

```json
{
  "id":    "cancel-1",
  "event": "cancel_order",
  "code":  200000,
  "msg":   "Success",
  "data": {
    "orderId":       "1234567890123456",
    "clientOrderId": "agent-001",
    "orderState":    "OPEN",
    "action":        "CANCEL_PENDING"
  }
}
```

> Cancel is asynchronous at the exchange layer. `CANCEL_PENDING` means accepted, not confirmed. Poll `rapidx order query` or listen to the `Orders` push channel to confirm the order reaches `CANCELLED`.
