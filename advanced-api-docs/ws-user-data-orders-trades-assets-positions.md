# User Data — Orders, Trades, Assets, Positions Push

**WebSocket URL:** Phase II (Mainnet): `wss://wss.liquiditytech.com/v1/private` · Phase I (Sandbox): `wss://wss.ltp-contest.com/v1/private`
> **Official API docs:** https://apidocliquidity.readme.io/reference/ws-user-data-orders-trades-assets-positions

After successful login, the server automatically pushes updates on three channels.

## Orders Channel

Pushed on every order state change.

```json
{
  "channel": "Orders",
  "data": {
    "orderId":          "1234567890123456",
    "clientOrderId":    "agent-001",
    "sym":              "BINANCE_PERP_BTC_USDT",
    "side":             "BUY",
    "positionSide":     "LONG",
    "orderType":        "LIMIT",
    "orderState":       "FILLED",
    "orderQty":         "0.001",
    "limitPrice":       "81000.0",
    "executedQty":      "0.001",
    "executedAvgPrice": "81000.0",
    "executedAmount":   "81.00",
    "fee":              "0.0324",
    "feeCoin":          "USDT"
  }
}
```

### Order State Values

| State | Meaning |
|-------|---------|
| `NEW` | Acknowledged by exchange |
| `OPEN` | Resting on the order book |
| `PARTIALLY_FILLED` | Partially executed |
| `FILLED` | Fully executed |
| `CANCELLED` | Cancelled |
| `REJECTED` | Rejected by exchange |

## Positions Channel

Pushed when a position changes (open, increase, decrease, or close).

```json
{
  "channel": "Positions",
  "data": {
    "sym":          "BINANCE_PERP_BTC_USDT",
    "positionSide": "LONG",
    "positionQty":  "0.001",
    "avgPrice":     "81000.0",
    "unrealizedPNL":"0.42",
    "leverage":     "5"
  }
}
```

## Assets Channel

Pushed when an account balance changes.

```json
{
  "channel": "Assets",
  "data": {
    "coin":    "USDT",
    "balance": "1000.000",
    "available":"950.000",
    "frozen":  "50.000"
  }
}
```

