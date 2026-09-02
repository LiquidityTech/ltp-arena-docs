# WebSocket — Cancel All Orders

**WebSocket URL:** Phase II (Mainnet): `wss://wss.liquiditytech.com/v1/private` · Phase I (Sandbox): `wss://wss.ltp-contest.com/v1/private`
> **Official API docs:** https://apidocliquidity.readme.io/reference/ws-cancel-orders

Cancel all open orders for a symbol or exchange.

## Request

```json
{
  "id": "cancelall-1",
  "action": "cancel_orders",
  "args": {
    "sym": "BINANCE_PERP_BTC_USDT"
  }
}
```

## Request Fields

| Field | Required | Description |
|-------|----------|-------------|
| `sym` | Conditional | Cancel all orders for this symbol (or use `exchangeType`) |
| `exchangeType` | Conditional | Cancel all orders on this exchange, e.g. `BINANCE` |

## Response (accepted)

```json
{
  "id":    "cancelall-1",
  "event": "cancel_orders",
  "code":  200000,
  "msg":   "Success",
  "data":  {}
}
```

