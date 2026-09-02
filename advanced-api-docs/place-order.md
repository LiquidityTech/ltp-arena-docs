# Place Order


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/place-order

**Rate limit**: 1 request / 5 s

## Endpoint

```
POST /api/v1/trading/order
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `clientOrderId` | string | No | Client-assigned order ID, max 40 chars; recommended for idempotency |
| `sym` | string | Yes | LTP symbol, e.g. `BINANCE_PERP_BTC_USDT` |
| `side` | string | Yes | `BUY` or `SELL` |
| `orderType` | string | Yes | `LIMIT` or `MARKET` |
| `timeInForce` | string | No | `GTC` (default) / `IOC` / `FOK` / `GTX` (post-only) |
| `orderQty` | string | No | Base/contract quantity. Binance: base asset amount; OKX: contract count |
| `limitPrice` | string | No | Limit price — required for `LIMIT` orders |
| `quoteOrderQty` | string | No | Quote-currency amount for SPOT MARKET BUY (alternative to `orderQty`) |
| `reduceOnly` | string | No | `true` to allow only position-reducing fills (perpetual only) |
| `positionSide` | string | No | `LONG` / `SHORT` — required in BOTH (hedge) position mode |
| `tpTriggerPrice` | string | No | Take-profit trigger price |
| `tpTriggerType` | string | No | Take-profit trigger type: `MarkPrice` / `LastPrice` |
| `tpPrice` | string | No | Take-profit order price (`0` = market) |
| `slTriggerPrice` | string | No | Stop-loss trigger price |
| `slTriggerType` | string | No | Stop-loss trigger type: `MarkPrice` / `LastPrice` |
| `slPrice` | string | No | Stop-loss order price (`0` = market) |

## Response (HTTP 200)

```json
{  
    "code": 200000,  
    "message": "Success",  
    "data": {  
        "orderId": "1234567890123456",
        "clientOrderId": "10003232424324324243243201"
    }  
}
```

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/order" \
  -X POST \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"sym":"BINANCE_PERP_BTC_USDT","side":"BUY","positionSide":"LONG","orderType":"LIMIT","timeInForce":"GTC","orderQty":"0.001","limitPrice":"60000","clientOrderId":"demo-001"}' | jq .
```
