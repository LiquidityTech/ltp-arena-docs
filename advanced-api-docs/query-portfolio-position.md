# Query Portfolio Position


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-portfolio-position

**Rate limit**: 60 requests per 60 seconds

## Endpoint

```
GET /api/v1/trading/position
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Filter by symbol (optional) |
| `exchange` | string | No | Filter by exchange (optional) |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": [
    {
      "positionId": "20164589748662850",
      "portfolioId": "2130592799975490",
      "sym": "BINANCE_PERP_ETH_USDT",
      "positionSide": "LONG",
      "positionMargin": "26.15922470925",
      "positionMM": "4.18547595348",
      "positionQty": "0.25",
      "positionValue": "523.184494185",
      "unrealizedPNL": "0.006994185",
      "unrealizedPNLRate": "0.000267373310205427",
      "avgPrice": "2092.71",
      "markPrice": "2092.73797674",
      "leverage": "20",
      "maxLeverage": "20",
      "riskLevel": "1",
      "fee": "0.18311213",
      "fundingFee": "0",
      "createAt": "1775010129278",
      "updateAt": "1775010134573",
      "liqPrice": "913.006664676013687927",
      "tpslOrder": []
    }
  ]
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

curl -s "${BASE}/api/v1/trading/position" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
