# BBO (Best Bid & Offer)

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-bbo

Pushed whenever the best bid or best offer price or quantity changes.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "BBO", "sym": "BINANCE_PERP_BTC_USDT" }]
}
```

## Push Message

```json
{
  "arg": { "channel": "BBO", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": {
    "bid":    "81422.3",
    "bidqty": "0.500",
    "ask":    "81422.4",
    "askqty": "0.300",
    "ts":     1719388800123
  }
}
```

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `bid` | string | Best bid price |
| `bidqty` | string | Best bid quantity |
| `ask` | string | Best ask price |
| `askqty` | string | Best ask quantity |
| `ts` | long | Server timestamp (epoch ms) |

