# Order Book

**WebSocket URL:** Phase II (Mainnet): `wss://md.liquiditytech.com/marketdata/v2/public` · Phase I (Sandbox): `wss://md.ltp-contest.com/marketdata/v2/public`
> **Official API docs:** https://apidocliquidity.readme.io/reference/market-data-order-book

Pushed every ~250 ms. First push is a full snapshot; subsequent pushes are incremental updates.

## Subscribe

```json
{
  "event": "subscribe",
  "arg": [{ "channel": "ORDER_BOOK", "sym": "BINANCE_PERP_BTC_USDT" }]
}
```

## Push Message (snapshot)

```json
{
  "arg": { "channel": "ORDER_BOOK", "sym": "BINANCE_PERP_BTC_USDT" },
  "data": {
    "updatetype": "snapshot",
    "seq":        1000,
    "Bids": [["81420.0", "1.500"], ["81419.0", "2.000"]],
    "Asks": [["81422.4", "0.800"], ["81423.0", "3.200"]]
  }
}
```

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `updatetype` | string | `snapshot` (first message) or `update` (incremental) |
| `seq` | long | Sequence number — discard messages with lower seq |
| `Bids` | array | `[price, qty]` pairs, best bid first |
| `Asks` | array | `[price, qty]` pairs, best ask first |

A `qty` of `"0"` in an incremental update means that price level should be removed.

