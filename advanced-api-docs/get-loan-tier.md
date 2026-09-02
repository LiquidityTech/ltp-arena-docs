# Get Loan Tier


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/get-loan-tier

**Rate limit**: 1 request per 20 seconds

## Endpoint

```
GET /api/v1/trading/loan/info_detail
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `exchangeType` | string | No | Exchange, e.g. `BINANCE` (optional) |
| `coin` | string | No | Coin symbol, e.g. `BTC` (optional) |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "BINANCE_MARGIN_AAVE_USDT": {
      "exchangeType": "BINANCE",
      "coin": "AAVE",
      "tiers": [
        {
          "tier": "1",
          "minSize": "0",
          "maxSize": "5400",
          "mmRate": "0.1000",
          "maxLeverage": "2.00"
        }
      ]
    },
    "OKX_MARGIN_SONIC_USDT": {
      "exchangeType": "OKX",
      "coin": "SONIC",
      "tiers": [
        {
          "tier": "1",
          "minSize": "0",
          "maxSize": "15000",
          "mmRate": "0.0600",
          "maxLeverage": "5.00"
        },
        {
          "tier": "2",
          "minSize": "15000",
          "maxSize": "25000",
          "mmRate": "0.1000",
          "maxLeverage": "5.00"
        }
      ]
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
PARAMS="exchangeType=BINANCE&coin=BTC"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/loan/info?exchangeType=BINANCE&coin=BTC" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
