# Close Positions


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/close-positions

**Rate limit**: 1 request / 10 s

## Endpoint

```
DELETE /api/v1/trading/positions
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `symList` | string | No | BINANCE_PERP_BTC_USDT,BINANCE_PERP_ETH_USDT |
| `positionSide` | string | No | LONG/SHORT |
| `closeAllPos` | string | No | true/false |
| `exchangeType` | string | No | Exchange to close all positions on (e.g. `BINANCE`) |

## Response (HTTP 202)

```json
{
  "code": 200000,
  "message": "Success",
  "data": [
    {
      "sym": "BINANCE_PERP_DOGE_USDT",
      "positionSide": "NONE",
      "orderId": "2154643761198660",
      "success": "true",
      "errorMsg": ""
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

curl -s "${BASE}/api/v1/trading/positions" \
  -X DELETE \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"exchangeType":"BINANCE","closeAllPos":"true"}' | jq .
```
