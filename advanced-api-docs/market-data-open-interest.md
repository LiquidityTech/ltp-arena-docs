# Open Interest

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-open-interest

Pushed when open interest changes. Available for perpetual symbols only.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "OPEN_INTEREST", "sym": "BINANCE_PERP_BTC_USDT" }]
}
```

## Push Message

```json
{
  "arg": { "channel": "OPEN_INTEREST", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": {
    "open_interest": "85432.120",
    "ts":            1719388800000
  }
}
```

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `open_interest` | string | Total open contracts (base currency) |
| `ts` | long | Server timestamp (epoch ms) |

