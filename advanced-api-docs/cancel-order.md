# Cancel Order


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/cancel-order

**Rate limit**: 1 request / 5 s

## Endpoint

```
DELETE /api/v1/trading/order
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `orderId` | string | No | Order ID |
| `clientOrderId` | string | No | Customer-defined order ID |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "orderId": "1234567890123456",
    "clientOrderId": "my-order-001",
    "orderState": "OPEN",
    "action": "CANCEL_PENDING",
    "actionMsg": ""
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
  -X DELETE \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"orderId":"1234567890123456"}' | jq .
```
