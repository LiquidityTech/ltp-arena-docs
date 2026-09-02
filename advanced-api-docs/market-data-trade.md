# Trade

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-trade

Pushed in real-time for each executed trade.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "TRADE", "sym": "BINANCE_PERP_BTC_USDT" }]
}
```

## Push Message

```json
{
  "arg": { "channel": "TRADE", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": [
    {
      "tradeid": "987654321",
      "side":    "BUY",
      "price":   "81422.4",
      "qty":     "0.002",
      "ts":      1719388800050
    }
  ]
}
```

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `tradeid` | string | Unique trade ID |
| `side` | string | `BUY` or `SELL` (taker side) |
| `price` | string | Trade price |
| `qty` | string | Trade quantity |
| `ts` | long | Trade timestamp (epoch ms) |

