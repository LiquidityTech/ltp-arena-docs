# Sym Info


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/sym-info

**Rate limit**: 3 requests / 10 s

## Endpoint

```
GET /api/v1/trading/sym/info
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Single symbol to query (optional — omit to return all) |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "BINANCE_PERP_IMX_USDT": {
      "sym": "BINANCE_PERP_IMX_USDT",
      "originalSymbol": "IMXUSDT",
      "state": "live",
      "pricePrecision": "4",
      "qtyPrecision": "0",
      "lotSize": "1",
      "tickSize": "0.0001",
      "minNotional": "5",
      "maxLimitSize": "5000000",
      "maxMarketSize": "500000",
      "maxNumOrders": "60",
      "minSize": "1",
      "contractSize": "1",
      "defaultLeverage": "5",
      "safeLeverage": "10",
      "liquidationFee": "0.030000"
    },
    "OKX_SPOT_WIF_USDT": {
      "sym": "OKX_SPOT_WIF_USDT",
      "originalSymbol": "WIF-USDT",
      "state": "live",
      "pricePrecision": "4",
      "qtyPrecision": "3",
      "lotSize": "0.001",
      "tickSize": "0.0001",
      "minNotional": "0",
      "maxLimitSize": "999999999999999",
      "maxMarketSize": "1000000",
      "maxNumOrders": "50",
      "minSize": "1",
      "defaultLeverage": "1",
      "safeLeverage": "1"
    }
  }
}
```

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
PARAMS="sym=BINANCE_PERP_BTC_USDT"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/sym/info?sym=BINANCE_PERP_BTC_USDT" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
