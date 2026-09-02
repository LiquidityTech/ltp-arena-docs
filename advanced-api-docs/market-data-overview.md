# Market Data WebSocket — Overview

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-overview

The market data WebSocket delivers real-time price and trade data. Append `?binary=false` to receive plain-text JSON instead of GZIP frames.

## Connection Limits

| Mode | Max connections / IP | Max symbols / connection |
|------|---------------------|--------------------------|
| Unauthenticated | 5 | 5 |
| Authenticated | 40 | 50 |

Limits are counted by **trading pair**, not by channel. Subscribing to BBO + TICKER + TRADE for the same symbol counts as 1 pair.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO",   "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TRADE", "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

## Subscription Confirmation

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "BBO", "sym": "BINANCE_PERP_BTC_USDT" }],
  "code": 0
}
```

## Available Channels

| Channel | Frequency | Scope |
|---------|-----------|-------|
| [`BBO`](market-data-bbo.md) | On change | All |
| [`TICKER`](market-data-ticker.md) | 2000 ms | All |
| [`TRADE`](market-data-trade.md) | Real-time | All |
| [`ORDER_BOOK`](market-data-order-book.md) | 250 ms | All |
| [`KLINE`](market-data-kline-candlestick.md) | Real-time | All |
| [`MARK_PRICE`](market-data-price.md) | Real-time | Perpetual only |
| [`INDEX_PRICE`](market-data-price.md) | Real-time | Perpetual only |
| [`MARK_PRICE_KLINE`](market-data-kline-candlestick.md) | Real-time | Perpetual only |
| [`INDEX_KLINE`](market-data-kline-candlestick.md) | Real-time | Perpetual only |
| [`OPEN_INTEREST`](market-data-open-interest.md) | On change | Perpetual only |

## Keepalive

Send `{ "ping": <timestamp_ms> }` every 20 seconds; server responds with `{ "pong": <timestamp_ms> }`.

## Error Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `11100` | Invalid symbol |
| `11210` | Unauthenticated symbol limit (5) exceeded |
| `11220` | Authenticated symbol limit (50) exceeded |
| `11250` | Symbol not currently supported |
| `11260` | Request rate limit exceeded |
