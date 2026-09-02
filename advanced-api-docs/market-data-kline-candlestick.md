# Kline / Candlestick

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-kline-candlestick

Supported channels: `KLINE`, `MARK_PRICE_KLINE`, `INDEX_KLINE`.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" },
    { "channel": "MARK_PRICE_KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "5m" }
  ]
}
```

### Supported intervals

`1m` `3m` `5m` `15m` `30m` `1h` `2h` `4h` `6h` `12h` `1d` `1w`

## Push Message

```json
{
  "arg": { "channel": "KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" },
  "data": [
    [1719388800000, "81000.0", "81500.0", "80900.0", "81422.4", "123.450", false]
  ]
}
```

## Kline Array Fields (index order)

| Index | Description |
|-------|-------------|
| `0` | Open time (epoch ms) |
| `1` | Open price |
| `2` | High price |
| `3` | Low price |
| `4` | Close price |
| `5` | Volume (base currency) |
| `6` | `true` if the candle is closed, `false` if still in progress |

> `MARK_PRICE_KLINE` and `INDEX_KLINE` are available for perpetual symbols only.

