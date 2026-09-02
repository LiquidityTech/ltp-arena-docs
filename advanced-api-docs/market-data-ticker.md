# Ticker (24-Hour Rolling Stats)

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-ticker

Pushed every ~2000 ms with 24-hour rolling statistics.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "TICKER", "sym": "BINANCE_PERP_BTC_USDT" }]
}
```

## Push Message

```json
{
  "arg": { "channel": "TICKER", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": {
    "last":   "81422.4",
    "chg":    "1.25",
    "high24h":"83000.0",
    "low24h": "79500.0",
    "vol24h": "15234.812",
    "ts":     1719388800000
  }
}
```

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `last` | string | Last trade price |
| `chg` | string | 24-hour price change percentage |
| `high24h` | string | 24-hour high price |
| `low24h` | string | 24-hour low price |
| `vol24h` | string | 24-hour volume (base currency) |
| `ts` | long | Server timestamp (epoch ms) |

