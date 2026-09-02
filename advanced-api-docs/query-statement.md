# Query Statement


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-statement

**Rate limit**: 20 requests per 60 seconds

## Endpoint

```
GET /api/v1/trading/statement
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `coin` | string | No |  |
| `statementType` | string | No |  |
| `exchange` | string | No |  |
| `startTime` | string | No |  |
| `endTime` | string | No |  |
| `page` | string | No |  |
| `pageSize` | string | No |  |
| `sym` | string | No | Filter by symbol (optional) |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "page": 1,
    "pageSize": 2,
    "pageNum": 212,
    "totalSize": 423,
    "list": [
      {
        "portfolioId": 2171869303228000,
        "requestId": "1754398800000",
        "statementId": "78472824993350212",
        "coin": "ETH",
        "sym": "",
        "statementType": "DEDUCT_INTEREST",
        "exchangeType": "OKX",
        "businessType": "SPOT",
        "beforeAvailable": "0",
        "afterAvailable": "0",
        "beforeOverdraw": "0.000000017314874246",
        "afterOverdraw": "0.000000023086525446",
        "beforeBorrow": "0",
        "afterBorrow": "0",
        "deltaAmount": "0",
        "createAt": 1754398800000
      },
      {
        "portfolioId": 1718693032228000,
        "requestId": "1754395200000",
        "statementId": "78457725511534148",
        "coin": "ETH",
        "sym": "",
        "statementType": "DEDUCT_INTEREST",
        "exchangeType": "OKX",
        "businessType": "SPOT",
        "beforeAvailable": "0",
        "afterAvailable": "0",
        "beforeOverdraw": "0.000000011543236272",
        "afterOverdraw": "0.000000017314874246",
        "beforeBorrow": "0",
        "afterBorrow": "0",
        "deltaAmount": "0",
        "createAt": 1754395200000
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

curl -s "${BASE}/api/v1/trading/statement" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
