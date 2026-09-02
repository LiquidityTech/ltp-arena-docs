# Query ADL Rank


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-adl-rank

**Rate limit**: 2 requests per 60 seconds

## Endpoint

```
GET /api/v1/adl/rank
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Filter by symbol (optional) |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "adlRanks": [
      {
        "sym": "BINANCE_PERP_BTC_USDT",
        "adlRank": "5",
        "positionId": "12345",
        "positionSide": "LONG"
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
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/adl/rank" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
