# Mark Price & Index Price

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-price

Available for perpetual symbols only. Used as the liquidation reference price.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "MARK_PRICE",  "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "INDEX_PRICE", "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

## Push Message — MARK_PRICE

```json
{
  "arg": { "channel": "MARK_PRICE", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": {
    "markpx": "81400.12",
    "ts":     1719388800000
  }
}
```

## Push Message — INDEX_PRICE

```json
{
  "arg": { "channel": "INDEX_PRICE", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": {
    "indexpx": "81398.75",
    "ts":      1719388800000
  }
}
```

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `markpx` | string | Current mark price (MARK_PRICE only) |
| `indexpx` | string | Current index price (INDEX_PRICE only) |
| `ts` | long | Server timestamp (epoch ms) |

