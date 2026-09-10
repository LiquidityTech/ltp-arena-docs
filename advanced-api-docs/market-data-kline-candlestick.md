# Kline (Candlestick)

**WebSocket URL:** `wss://mds.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-kline-candlestick

Real-time OHLCV candlestick data.

Three channels are available depending on the price source. All support the same set of intervals.

| Channel | Price Source | Scope |
|---------|-------------|-------|
| `KLINE` | Trade price | All |
| `INDEX_KLINE` | Index price | Futures only |
| `MARK_PRICE_KLINE` | Mark price | Futures only |

**Supported intervals:** `1m` `3m` `5m` `15m` `30m` `1H` `2H` `4H` `8H` `12H` `1D` `2D` `5D` `1W` `1M`

### Request Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `sym` | Yes | Trading pair |
| `interval` | Yes | Candlestick interval (see supported values above) |

## KLINE

### Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" }]
}
```

### Push Data

```json
{
  "arg": { "channel": "KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" },
  "data": [["1774331880000", "70237.2", "70245.7", "70226.4", "70245.6", "26.861"]]
}
```

Array format: `[timestamp, open, high, low, close, volume]`

## INDEX_KLINE

### Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "INDEX_KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" }]
}
```

### Push Data

```json
{
  "arg": { "channel": "INDEX_KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" },
  "data": [["1774331940000", "70278.36", "70279.40", "70278.36", "70279.39", "0"]]
}
```

Array format: `[timestamp, open, high, low, close, volume]`

> `volume` is always `"0"` for `INDEX_KLINE` and `MARK_PRICE_KLINE` — these are derived price series with no trading volume.

## MARK_PRICE_KLINE

### Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "MARK_PRICE_KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" }]
}
```

### Push Data

```json
{
  "arg": { "channel": "MARK_PRICE_KLINE", "sym": "BINANCE_PERP_BTC_USDT", "interval": "1m" },
  "data": [["1774332000000", "70249.6", "70261.3", "70249.6", "70261.3", "0"]]
}
```

Array format: `[timestamp, open, high, low, close, volume]`

> `volume` is always `"0"` for `INDEX_KLINE` and `MARK_PRICE_KLINE` — these are derived price series with no trading volume.
