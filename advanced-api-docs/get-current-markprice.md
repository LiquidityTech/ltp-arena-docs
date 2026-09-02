# Get Current Mark Price


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/get-current-markprice

**Rate limit**: 3 requests / 10 s

## Endpoint

```
GET /api/v1/market/markPrice
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | Yes | Sym |

## Response (HTTP 200)

```json
{
    "code": 200000,
    "message": "Success",
    "data": [
        {
            "publicChannelTypeEnum": "MARK_PRICE",
            "sym": "BINANCE_PERP_ETH_USDT",
            "markPrice": "3289.00425758",
            "time": 1722396532000
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
PARAMS="sym=BINANCE_PERP_BTC_USDT"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/market/markPrice?sym=BINANCE_PERP_BTC_USDT" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
