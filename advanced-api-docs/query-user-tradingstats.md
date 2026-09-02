# Query User Trading Volume


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-user-tradingstats

**Rate limit**: 4 requests per second

## Endpoint

```
GET /api/v1/trading/user/tradingStats
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `exchange` | string | No |  |
| `businessType` | string | No |  |
| `begin` | string | Yes | Start timestamp (Unix seconds) |
| `endTime` | string | Yes |  |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "begin": "202411332",
    "end": "20250127",
    "allSpot": "0",
    "allPerp": "4687.3585",
    "details": [
      {
        "exchange": "BINANCE",
        "businessType": "PERP",
        "executedAmount": "3761.2156"
      },
      {
        "exchange": "BINANCE",
        "businessType": "SPOT",
        "executedAmount": "0"
      },
      {
        "exchange": "OKX",
        "businessType": "PERP",
        "executedAmount": "926.1429"
      }
    ]
  }
}
```

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
PARAMS="begin=1719302400&end=1719388800"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/user/tradingStats?begin=1719302400&end=1719388800" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
