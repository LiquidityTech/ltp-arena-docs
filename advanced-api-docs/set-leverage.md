# Set Leverage


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/set-leverage

**Rate limit**: 6 requests per second

## Endpoint

```
POST /api/v1/trading/position/leverage
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | Yes | Symbol (perpetual only) |
| `leverage` | string | Yes | Integer leverage value |

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

curl -s "${BASE}/api/v1/trading/position/leverage" \
  -X POST \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" \
  -d '{"sym":"BINANCE_PERP_BTC_USDT","leverage":"5"}' | jq .
```
