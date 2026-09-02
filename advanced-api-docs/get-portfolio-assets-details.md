# Get Portfolio Assets Details


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/get-portfolio-assets-details

**Rate limit**: 4 requests per second

## Endpoint

```
GET /api/v1/trading/portfolio/assets
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `exchangeType` | string | No |  |
| `page` | string | No |  |
| `pageSize` | string | No |  |

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
        "portfolioId": "17xxxxxxxxxxxxxx",
        "coin": "USDT",
        "exchangeType": "BINANCE",
        "balance": "33.970877268901446313",
        "available": "33.970877268901446313",
        "frozen": "0",
        "debt": "0",
        "equity": "33.970877268901446313",
        "createAt": "1776008971034",
        "updateAt": "1776008971034",
        "borrow": "0",
        "overdraw": "0",
        "indexPrice": "1",
        "marginValue": "33.9674801811745561683687",
        "virtualBorrow": "0",
        "upnl": "0",
        "debtMargin": "0",
        "perpMargin": "0",
        "maxTransferable": "33.970877268901446313",
        "equityValue": "33.970877268901446313"
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

curl -s "${BASE}/api/v1/trading/portfolio/assets" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
