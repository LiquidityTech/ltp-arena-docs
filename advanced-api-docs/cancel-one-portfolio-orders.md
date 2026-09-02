# Cancel Orders


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/cancel-one-portfolio-orders

**Rate limit**: 10 requests per 10 seconds

## Endpoint

```
DELETE /api/v1/trading/cancelAll
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `exchangeType` | string | No | Cancel all orders on this exchange (e.g. `BINANCE`) |
| `sym` | string | No | Cancel all orders for this symbol |

## Response (HTTP 202)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {}
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

curl -s "${BASE}/api/v1/trading/cancelAll" \
  -X DELETE \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"sym":"BINANCE_PERP_BTC_USDT"}' | jq .
```
