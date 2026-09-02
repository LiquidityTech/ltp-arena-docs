# Replace Order


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/replace-order

**Rate limit**: 1 request / 5 s

## Endpoint

```
PUT /api/v1/trading/order
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `orderId` | string | Yes | order ID |
| `replaceQty` | string | No | replace order qty |
| `replacePrice` | string | No | replace order price |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "orderId": "1722413861764000",
    "orderState": "OPEN"
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

curl -s "${BASE}/api/v1/trading/order" \
  -X PUT \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"orderId":"1234567890123456","replacePrice":"62000"}' | jq .
```
