# Query Portfolio History Position


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-portfolio-history-position

**Rate limit**: 6 requests per 10 seconds

## Endpoint

```
GET /api/v1/trading/history/position
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Filter by symbol (optional) |
| `exchange` | string | No |  |
| `begin` | string | No |  |
| `end` | string | No |  |
| `page` | string | No | Page number |
| `pageSize` | string | No | Items per page |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "page": 1,
    "pageSize": 1000,
    "pageNum": 1,
    "totalSize": 1,
    "list": [
      {
        "positionId": "20168941947142980",
        "portfolioId": "17xxxxxxxxxxxxxx",
        "portfolioName": null,
        "sym": "BINANCE_PERP_BTC_USDT",
        "closedType": "COMPLETE_CLOSED",
        "closedPnl": "-0.0066",
        "closedPnlRatio": "-0.00411522633744856",
        "closedAvgPrice": "1.214",
        "maxPositionQty": "6.6",
        "closedQty": "6.6",
        "liqFee": "0",
        "fundingFee": "0",
        "fee": "0.0028055",
        "openAvgPrice": "1.215",
        "leverage": 5,
        "positionHistorySide": "LONG",
        "positionMode": "NET",
        "createAt": "1775987218376",
        "updateAt": "1775987219638"
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

curl -s "${BASE}/api/v1/trading/history/position" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
