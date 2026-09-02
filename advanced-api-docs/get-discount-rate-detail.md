# Get Discount Rate


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/get-discount-rate-detail

**Rate limit**: 1 request per 20 seconds

## Endpoint

```
GET /api/v1/trading/coin/discount_detail
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `coin` | string | No | Coin |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "BTC": {
      "coin": "BTC",
      "discount": "0.9500"
    },
    "APT": {
      "coin": "APT",
      "discount": "0.5000"
    },
    "RAY": {
      "coin": "RAY",
      "discount": "0.3000"
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
PARAMS="coin=BTC"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/coin/discount?coin=BTC" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
