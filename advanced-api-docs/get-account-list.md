# Get Account List


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/get-account-list

**Rate limit**: not specified in official docs — apply conservative limits

## Endpoint

```
GET /api/v1/tradeAccount/list
```

## Response (HTTP 200)

```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "accountId": 12345,
      "venues": "BINANCE",
      "label": "DMA",
      "exchangeUid": 123456789,
      "accountNote": "my-binance-account"
    },
    {
      "accountId": 22222,
      "venues": "OKEX",
      "label": "DMA",
      "exchangeUid": 0,
      "accountNote": ""
    },
    {
      "accountId": 33333,
      "venues": "LTP",
      "label": "Funding Account",
      "exchangeUid": 0,
      "accountNote": ""
    },
    {
      "accountId": 44444,
      "venues": "CL Trading Account",
      "label": "ClearLoop",
      "exchangeUid": 0,
      "accountNote": ""
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

curl -s "${BASE}/api/v1/trading/account" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
